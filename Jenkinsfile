pipeline {
     agent none

    options {
        skipDefaultCheckout()
    }
    stages {
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
            agent {
                label 'Second'
            }

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

        // 2. Останавливаем службу
        stage('Stop Service') {
            agent {
                label 'Second'
            }

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

        // 3. Очищаем кэш службы печати
        stage('Clean Cache') {
            agent {
                label 'Second'
            }

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

        // 4. Перезагружаем виртуальную машину
        stage('Restart VM') {
            agent {
                label 'Second'
            }

            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Write-Host "Restarting Windows VM..."

                    Start-Process `
                        -FilePath "$env:SystemRoot\\System32\\shutdown.exe" `
                        -ArgumentList "/r /t 30 /f" `
                        -WindowStyle Hidden

                    Write-Host "Restart command sent successfully."
                '''
            }
        }

        // 5. Ждём возвращения агента и проверяем службу
        stage('Verify Service') {
            agent {
                label 'Second'
            }

            options {
                timeout(time: 15, unit: 'MINUTES')
            }

            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    $service = Get-Service -Name "Spooler" -ErrorAction Stop

                    Write-Host "Checking service after reboot..."
                    Write-Host "Current status: $($service.Status)"

                    if ($service.Status -ne "Running") {
                        throw "ERROR: Spooler is not running after reboot!"
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