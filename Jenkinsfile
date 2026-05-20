pipeline {
    agent any

    stages {
        stage('Validar Inicio') {
            steps {
                echo 'El servidor EC2 ha enviado la señal de arranque con éxito.'
            }
        }
        
        stage('Notificar a Slack') {
            steps {
                // Esta función asume que ya configuraste el plugin y las credenciales en Jenkins
                slackSend(
                    channel: '#general', // Cambia esto si usas otro canal
                    color: 'good',
                    message: "🚀 ¡Alerta de Infraestructura! El servidor EC2 acaba de encenderse y Jenkins está operativo."
                )
            }
        }
    }
}