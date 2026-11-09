
pipeline {
    agent none

    options {
        skipDefaultCheckout()
    }

    stages {

        stage('Check Admin Rights') {
            agent { label 'Second' }
            steps {
                powershell '''
                    Write-Host "Current user:"
                    whoami

                    Write-Host "Administrator status:"
                    $identity = [Security.Principal.WindowsIdentity]::GetCurrent()
                    $principal = New-Object Security.Principal.WindowsPrincipal($identity)

                    $isAdmin = $principal.IsInRole(
                        [Security.Principal.WindowsBuiltInRole]::Administrator
                    )

                    Write-Host $isAdmin
                    Write-Host "User groups:"
                    whoami /groups

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
                echo 'Checking target VM...'
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
                    Write-Host "Service Name: $($service.Name)"
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

                    Write-Host "Restarting Windows VM..."

                    Start-Process `
                        -FilePath "$env:SystemRoot\\System32\\shutdown.exe" `
                        -ArgumentList "/r /t 5 /f" `
                        -WindowStyle Hidden

                    Write-Host "Restart command sent successfully."

                    Start-Sleep -Seconds 2
                '''
            }
        }

        
stage('Verify Service') {
    agent { label 'Second' }

    options {
        timeout(time: 5, unit: 'MINUTES')
    }

    steps {
        powershell '''
            $ErrorActionPreference = "Stop"

            Write-Host "Agent is back online."
            Write-Host "Waiting for Spooler to start..."

            $deadline = (Get-Date).AddMinutes(3)
            $isRunning = $false

            while ((Get-Date) -lt $deadline) {
                $service = Get-Service -Name "Spooler"

                Write-Host "Current status: $($service.Status)"

                if ($service.Status -eq "Running") {
                    $isRunning = $true
                    break
                }

                Start-Sleep -Seconds 5
            }

            if (-not $isRunning) {
                throw "ERROR: Spooler did not start within 3 minutes!"
            }

            Write-Host "SUCCESS: Spooler is running!"
            Write-Host "SERVICE CHECK PASSED"
        '''
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
