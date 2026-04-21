# 📌 AuthApp CI

Pipeline de integración continua para una aplicación Java utilizando Jenkins, Maven y SonarQube. El proyecto analiza el código automáticamente con cada cambio en GitHub, compila, ejecuta pruebas y envía los resultados a SonarQube para revisión de calidad.

## 🧱 Composición del Proyecto

- 🔧 **Java 21** — Código fuente de la app (`/authapp`)
- ⚙️ **Maven** — Build y ejecución de pruebas
- 🚀 **Jenkins (Docker)** — Automatiza el proceso CI/CD
- 📊 **SonarQube (Docker)** — Análisis estático de código
- 🐳 **Docker Compose** — Orquestación de servicios

## 🖼️ Vista Previa

| Jenkins                     | SonarQube                       |
| --------------------------- | ------------------------------- |
| ![jenkins](img/jenkins.png) | ![sonarqube](img/sonarqube.png) |

## ⚙️ Requisitos

- Docker
- Docker Compose
- Git

## 🚀 Instalación

```bash
# Clona el repositorio
git clone https://github.com/brayandiazc/authapp-ci.git
cd authapp-ci

# Asegúrate de tener esta estructura:
# .
# ├── docker-compose.yml
# ├── jenkins/
# │   └── Dockerfile
# └── authapp/
#     └── (código Java + pom.xml)

# Construye y levanta los contenedores
docker compose up --build -d

# Extrae la contraseña inicial de Jenkins
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

## 🔐 Configuración de Jenkins

1. Accede a [http://localhost:8080](http://localhost:8080)

2. Ingresa la contraseña inicial (ver paso anterior)

3. Instala los plugins sugeridos

4. Configura las herramientas:

   - **Java 21** (verifica con `readlink -f $(which java)`)
   - **Maven 3.x** (`/usr/share/maven`)
   - **SonarQube**:

     - Nombre: `SonarQube`
     - URL: `http://sonarqube:9000`

5. Agrega un nuevo _Pipeline Job_:

   - Origen: Git → `https://github.com/brayandiazc/authapp-ci.git`
   - Branch: `main`
   - Script Path: `Jenkinsfile`

## 📄 Jenkinsfile

```groovy
pipeline {
  agent any

  tools {
    jdk 'Java21'
    maven 'Maven3'
  }

  environment {
    SONARQUBE_ENV = 'SonarQube'
  }

  stages {
    stage('Build y Test') {
      steps {
        dir('authapp') {
          sh 'mvn clean test'
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("${SONARQUBE_ENV}") {
          dir('authapp') {
            sh '''
              mvn sonar:sonar \
                -Dsonar.projectKey=authapp \
                -Dsonar.host.url=http://sonarqube:9000 \
                -Dsonar.login=tu_token_aqui
            '''
          }
        }
      }
    }
  }
}
```

🔐 **Importante**: genera tu token en SonarQube:
`http://localhost:9000 > My Account > Security > Generate Tokens`

## 🧪 Pruebas

El sistema ejecuta automáticamente `mvn clean test` dentro del pipeline CI.

## 🛑 Parar el entorno

```bash
docker compose down
```

Para limpiar todo:

```bash
docker compose down -v --remove-orphans
```

## 🛣️ Roadmap

- [x] Jenkins con Maven
- [x] Análisis con SonarQube
- [ ] Notificaciones Slack
- [ ] Despliegue automático (CD)

## 🖇️ Contribuye

```bash
# Fork → Rama → Cambios → Pull Request
```

## 📄 Licencia

MIT — ver [LICENSE](LICENSE.md)

⌨️ con ❤️ por [Brayan Diaz C](https://github.com/brayandiazc)
