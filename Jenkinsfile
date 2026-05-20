pipeline {
    agent any

    // Variables globales para usar en los mensajes
    environment {
        APP_IMAGE = 'mi-empresa/backend:v2.4'
        AMBIENTE = 'Producción'
        SLACK_DEPLOYMENTS = '#deployments'
        SLACK_INCIDENTS = '#incidents'
    }

    stages {
        stage('Análisis de Seguridad (CVE)') {
            steps {
                script {
                    // Simulamos que encontramos un error crítico
                    def cveCritico = false 
                    if (cveCritico) {
                        slackSend(
                            channel: env.SLACK_INCIDENTS,
                            color: 'danger',
                            message: "🚨 *CVE Crítico Detectado* 🚨\n*Imagen:* ${env.APP_IMAGE}\n*Ambiente:* ${env.AMBIENTE}\nNo se puede continuar con el despliegue."
                        )
                        error("Fallo por seguridad") // Esto detiene el pipeline
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    def qualityGatePass = true
                    if (!qualityGatePass) {
                        slackSend(
                            channel: env.SLACK_INCIDENTS,
                            color: 'warning',
                            message: "⚠️ *Quality Gate Fallido*\n*Imagen:* ${env.APP_IMAGE}\nEl código no cumple con los estándares mínimos de calidad."
                        )
                        error("Fallo de Quality Gate")
                    }
                }
            }
        }

        stage('Aprobación Manual') {
            steps {
                slackSend(
                    channel: env.SLACK_DEPLOYMENTS,
                    color: 'warning',
                    message: "⏳ *Esperando Aprobación*\n*Imagen:* ${env.APP_IMAGE}\n*Ambiente:* ${env.AMBIENTE}\nPor favor, apruebe en Jenkins para desplegar."
                )
                // Esto pausa el pipeline hasta que alguien le dé "Proceed" en Jenkins
                input message: '¿Aprobar pase a Producción?' 
            }
        }

        stage('Deploy') {
            steps {
                echo "Desplegando la aplicación..."
                // Lógica de despliegue aquí
            }
        }
    }

    // El bloque 'post' se ejecuta siempre al final, dependiendo de cómo le fue al pipeline
    post {
        success {
            slackSend(
                channel: env.SLACK_DEPLOYMENTS,
                color: 'good',
                message: "✅ *Deploy Exitoso*\n*Imagen:* ${env.APP_IMAGE}\n*Ambiente:* ${env.AMBIENTE}\n*Resultado:* Todo funcionando correctamente."
            )
        }
        failure {
            slackSend(
                channel: env.SLACK_DEPLOYMENTS,
                color: 'danger',
                message: "❌ *Pipeline Fallido*\n*Imagen:* ${env.APP_IMAGE}\n*Ambiente:* ${env.AMBIENTE}\nRevisa los logs aquí: ${env.BUILD_URL}"
            )
        }
        aborted {
            // Si alguien cancela el pipeline o se hace Rollback
            slackSend(
                channel: env.SLACK_INCIDENTS,
                color: '#FF00FF', // Morado
                message: "🔥 *Rollback / Cancelación Ejecutada*\n*Imagen:* ${env.APP_IMAGE}\n*Ambiente:* ${env.AMBIENTE}"
            )
        }
    }
}