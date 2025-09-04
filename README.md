# GitHub to GitLab Sync Lambda

Esta Lambda de AWS se activa automáticamente cuando se reciben webhooks de GitHub y sincroniza los cambios con un repositorio de GitLab, funcionando como un mirror automático.

## 🚀 Características

- **TypeScript**: Código completamente tipado con las últimas versiones de las librerías
- **Webhook de GitHub**: Se activa automáticamente con cada push al repositorio
- **Sincronización automática**: Clona el repositorio de GitHub y lo sincroniza con GitLab
- **Autenticación SSH**: Utiliza claves SSH para acceder a GitLab de forma segura
- **Filtrado de ramas**: Solo procesa cambios en las ramas `main` o `master`
- **Verificación de seguridad**: Valida la firma del webhook de GitHub
- **Limpieza automática**: Elimina archivos temporales después de la sincronización
- **Serverless Framework**: Configuración moderna con TypeScript y esbuild

## 🏗️ Arquitectura

```
GitHub Push → Webhook → API Gateway → Lambda → Git Operations → GitLab
```

## 📋 Prerrequisitos

- AWS CLI configurado
- Node.js 18.16+ instalado
- Serverless Framework instalado globalmente
- Acceso a GitLab con clave SSH
- Repositorio en GitLab creado previamente
- Git Layer configurada en AWS (para comandos Git en Lambda)

## 🛠️ Instalación

### 1. Clonar e instalar dependencias

```bash
git clone <tu-repositorio>
cd github-to-gitlab-sync
npm install
```

### 2. Configurar Git Layer (Requerido)

```bash
# Configurar layer de Git para Lambda
./scripts/setup-git-layer.sh
```

### 3. Configurar parámetros en AWS Systems Manager (SSM)

```bash
# Usar el script automatizado (recomendado)
./scripts/setup-ssm.sh

# O configurar manualmente
aws ssm put-parameter \
  --name "/github-to-gitlab/gitlab-ssh-key" \
  --value "$(cat ~/.ssh/id_rsa)" \
  --type "SecureString" \
  --region us-east-1

aws ssm put-parameter \
  --name "/github-to-gitlab/gitlab-url" \
  --value "git@gitlab.com" \
  --type "String" \
  --region us-east-1

aws ssm put-parameter \
  --name "/github-to-gitlab/github-webhook-secret" \
  --value "tu-secreto-seguro-aqui" \
  --type "SecureString" \
  --region us-east-1

aws ssm put-parameter \
  --name "/github-to-gitlab/gitlab-username" \
  --value "tu-usuario-gitlab" \
  --type "String" \
  --region us-east-1

aws ssm put-parameter \
  --name "/github-to-gitlab/gitlab-email" \
  --value "tu-email@ejemplo.com" \
  --type "String" \
  --region us-east-1
```

### 4. Desplegar la Lambda

```bash
# Desplegar en desarrollo
npm run deploy-dev

# Desplegar en staging
npm run deploy-staging

# Desplegar en producción
npm run deploy-prod

# O usar el script
./scripts/deploy.sh dev
```

## 🔧 Configuración del Webhook de GitHub

### 1. Ir a tu repositorio de GitHub
### 2. Settings → Webhooks → Add webhook
### 3. Configurar:
   - **Payload URL**: `https://tu-api-gateway-url/dev/webhook/github`
   - **Content type**: `application/json`
   - **Secret**: El mismo valor que configuraste en SSM
   - **Events**: Solo `Push events`
   - **Active**: ✅

## 🔑 Configuración de GitLab

### 1. Generar clave SSH

```bash
ssh-keygen -t rsa -b 4096 -C "tu-email@ejemplo.com"
```

### 2. Agregar la clave pública a GitLab
   - Ve a GitLab → User Settings → SSH Keys
   - Copia el contenido de `~/.ssh/id_rsa.pub`
   - Pega y guarda la clave

### 3. Crear el repositorio en GitLab
   - Crea un nuevo proyecto en GitLab
   - El nombre debe coincidir con el repositorio de GitHub

## 📝 Variables de Entorno

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `GITLAB_SSH_KEY` | Clave SSH privada para GitLab | Contenido de `~/.ssh/id_rsa` |
| `GITLAB_URL` | URL de GitLab | `git@gitlab.com` |
| `GITHUB_WEBHOOK_SECRET` | Secreto del webhook de GitHub | `mi-secreto-super-seguro` |
| `GITLAB_USERNAME` | Usuario de GitLab | `mi-usuario` |
| `GITLAB_EMAIL` | Email de GitLab | `mi-email@gitlab.com` |

## 🧪 Pruebas Locales

```bash
# Ejecutar serverless offline
npm run offline

# En otra terminal, probar el webhook
./scripts/test-webhook.sh
```

## 📊 Monitoreo

### Ver logs de la Lambda

```bash
npm run logs
```

### Ver logs en CloudWatch

```bash
aws logs tail /aws/lambda/github-to-gitlab-sync-dev --follow
```

## 🚨 Troubleshooting

### Error: "git: command not found"
- **Problema**: Git no está disponible en Lambda
- **Solución**: Ejecuta `./scripts/setup-git-layer.sh` para configurar la Git Layer

### Error: "Parameter not found"
- Ejecuta `./scripts/setup-ssm.sh` primero

### Error: "Permission denied"
- Verifica que tu clave SSH esté en GitLab
- Asegúrate de que el repositorio exista en GitLab

### Error: "Invalid webhook signature"
- Verifica que el secreto coincida entre GitHub y SSM

### Error: "Timeout"
- Aumenta el timeout en `serverless.ts` si es necesario
- Verifica que la Lambda tenga suficiente memoria asignada

## 🔒 Seguridad

- **Webhook verification**: Se valida la firma de GitHub para prevenir ataques
- **SSH keys**: Las claves SSH se almacenan de forma segura en SSM
- **TypeScript**: Código tipado para mayor seguridad y mantenibilidad
- **IAM**: Permisos mínimos necesarios para la operación

## 📈 Escalabilidad

- **Concurrent executions**: La Lambda puede manejar múltiples webhooks simultáneamente
- **Memory**: Configurada con 1GB de memoria para operaciones Git eficientes
- **Timeout**: 5 minutos para repositorios grandes
- **esbuild**: Bundling optimizado para mejor rendimiento

## 🏗️ Estructura del Proyecto

```
src/
├── functions/
│   └── githubToGitlabSync/
│       ├── handler.ts      # Función principal
│       └── index.ts        # Configuración de la función
├── libs/
│   ├── handler-resolver.ts # Resolver de paths
│   └── lambda.ts           # Middleware utilities
├── types/
│   └── index.ts            # Definiciones de tipos TypeScript
└── utils/
    ├── crypto.ts            # Utilidades de criptografía
    └── git.ts               # Operaciones Git
```

## 🚀 Scripts Disponibles

- `./scripts/setup-ssm.sh` - Configuración automática de SSM
- `./scripts/deploy.sh` - Despliegue automatizado
- `./scripts/test-webhook.sh` - Pruebas del webhook
- `./scripts/cleanup.sh` - Limpieza de recursos

## 🤝 Contribución

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

## 📄 Licencia

Este proyecto está bajo la Licencia MIT - ver el archivo [LICENSE](LICENSE) para detalles.

## 🆘 Soporte

Si tienes problemas o preguntas:

1. Revisa la sección de troubleshooting
2. Verifica los logs de CloudWatch
3. Abre un issue en GitHub con detalles del error
4. Incluye logs, configuración y pasos para reproducir el problema
# repoa
