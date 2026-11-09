
pipeline {
    agent none

    options {
        skipDefaultCheckout()
    }
    triggers {
        cron('0 7 * * *')

    stages {
        stage('Check Admin Rights') {
            agent { label 'Second' }
            steps {
                powershell '''
                    $identity = [Security.Principal.WindowsIdentity]::GetCurrent()
                    $principal = New-Object Security.Principal.WindowsPrincipal($identity)
                    $isAdmin = $principal.IsInRole(
                        [Security.Principal.WindowsBuiltInRole]::Administrator
                    )

                    Write-Host "Current user:"
                    whoami
                    Write-Host "Administrator status: $isAdmin"

                    if (-not $isAdmin) {
                        throw "Agent does not have Administrator privileges!"
                    }
                '''
            }
        }

        stage('Hello') {
            agent { label 'Second' }
            steps {
                echo 'Hello from Jenkins!'
            }
        }

        stage('Check Environment') {
            agent { label 'Second' }
            steps {
                bat 'whoami'
                bat 'java -version'
                bat 'git --version'
            }
        }

        stage('Build') {
            agent { label 'Second' }
            steps {
                echo 'Build completed successfully!'
            }
        }

        stage('Check Git') {
            agent { label 'Second' }
            steps {
                checkout scm
                bat 'git status'
                bat 'git branch'
            }
        }

        stage('Check VM') {
            agent { label 'Second' }
            steps {
                bat 'hostname'
                bat 'whoami'
            }
        }

        stage('Check Service') {
            agent { label 'Second' }
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"
                    $service = Get-Service -Name "Spooler" -ErrorAction Stop
                    Write-Host "Service found: $($service.DisplayName)"
                    Write-Host "Current Status: $($service.Status)"
                '''
            }
        }

        stage('Stop Service') {
            agent { label 'Second' }
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Set-Service -Name "Spooler" -StartupType Automatic
                    $service = Get-Service -Name "Spooler"

                    if ($service.Status -ne "Stopped") {
                        Stop-Service -Name "Spooler" -Force
                        $service.WaitForStatus(
                            [System.ServiceProcess.ServiceControllerStatus]::Stopped,
                            [TimeSpan]::FromSeconds(30)
                        )
                    }

                    $service = Get-Service -Name "Spooler"

                    if ($service.Status -ne "Stopped") {
                        throw "Service failed to stop!"
                    }

                    Write-Host "SUCCESS: Spooler is stopped."
                '''
            }
        }

        stage('Clean Cache') {
            agent { label 'Second' }
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    $service = Get-Service -Name "Spooler"
                    if ($service.Status -ne "Stopped") {
                        throw "Spooler must be stopped before cache cleanup!"
                    }

                    $cachePath = "$env:SystemRoot\\System32\\spool\\PRINTERS"

                    if (-not (Test-Path $cachePath)) {
                        throw "Spooler cache directory not found!"
                    }

                    $files = @(Get-ChildItem -Path $cachePath -Force -File)

                    if ($files.Count -gt 0) {
                        $files | Remove-Item -Force
                    }

                    Write-Host "SUCCESS: Spooler cache cleaned."
                    Write-Host "Deleted files: $($files.Count)"
                '''
            }
        }

        stage('Restart VM') {
    agent { label 'Second' }

    steps {
        powershell '''
            $ErrorActionPreference = "Stop"

            $bootTime = (Get-CimInstance Win32_OperatingSystem).LastBootUpTime
            $bootTime | Set-Content "$env:WORKSPACE\\boot-before.txt"

            Write-Host "Previous boot time: $bootTime"
            Write-Host "Restarting Windows VM..."

            Start-Process `
                -FilePath "$env:SystemRoot\\System32\\shutdown.exe" `
                -ArgumentList "/r /t 5 /f" `
                -WindowStyle Hidden

            Write-Host "Restart command sent."
        '''
    }
}

stage('Wait for VM Shutdown') {
    agent none

    steps {
        echo 'Waiting for Windows VM to shut down...'
        sleep time: 30, unit: 'SECONDS'
        echo 'Waiting period completed.'
    }
}
        stage('Verify Service') {
    agent { label 'Second' }

    options {
        timeout(time: 10, unit: 'MINUTES')
    }

    steps {
        powershell '''
            $ErrorActionPreference = "Stop"

            Write-Host "Agent reconnected."

            $beforeFile = "$env:WORKSPACE\\boot-before.txt"

            if (-not (Test-Path $beforeFile)) {
                throw "Previous boot time file not found!"
            }

            $previousBoot = [DateTime]::Parse(
                (Get-Content $beforeFile -Raw).Trim()
            )

            $currentBoot = (Get-CimInstance Win32_OperatingSystem).LastBootUpTime

            Write-Host "Previous boot: $previousBoot"
            Write-Host "Current boot: $currentBoot"

            if ($currentBoot -le $previousBoot.AddSeconds(30)) {
    throw "VM reboot was not detected or boot time did not change enough!"
}

            Write-Host "SUCCESS: VM reboot confirmed."

            $deadline = (Get-Date).AddMinutes(3)
            $isRunning = $false

            while ((Get-Date) -lt $deadline) {
                $service = Get-Service -Name "Spooler"

                Write-Host "Spooler status: $($service.Status)"

                if ($service.Status -eq "Running") {
                    $isRunning = $true
                    break
                }

                Start-Sleep -Seconds 5
            }

            if (-not $isRunning) {
                throw "Spooler did not start automatically!"
            }

            Write-Host "SUCCESS: Spooler is running!"
            Write-Host "SERVICE CHECK PASSED"
        '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: All operations completed successfully!'
        }

        failure {
            echo 'FAILED: One or more operations failed. Check Console Output.'
        }
    }
}
