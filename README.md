# Integración con Chatwoot

Este repositorio contiene la integración de un bot con [Chatwoot](https://www.chatwoot.com/), una plataforma de mensajería y soporte al cliente, utilizando un entorno basado en Docker. La integración permite procesar y almacenar mensajes de WhatsApp en Chatwoot. A continuación, se detallan los pasos para instalar, configurar y verificar el proyecto.

## Requisitos Previos

Antes de comenzar, asegúrate de contar con lo siguiente:

- **Docker y Docker Compose instalados**: Necesarios para ejecutar los contenedores de la aplicación.
- **Acceso a la interfaz de administración de Chatwoot**: Requerido para generar claves API y configurar inboxes.
- **Archivo `.env` con variables de entorno**: Consulta el archivo `.env.example` en el repositorio como referencia.

## Instrucciones de Instalación y Configuración

### 1. Levantar el Proyecto

Para iniciar el proyecto en modo producción, ejecuta el siguiente comando con el archivo de configuración de Docker Compose:

```bash
docker-compose -f docker-compose.production.yaml up -d
```

- **Explicación**:  
  El parámetro `-d` ejecuta los contenedores en segundo plano, liberando el terminal. Asegúrate de que el archivo `docker-compose.production.yaml` esté ubicado en el directorio raíz del proyecto.

### 2. Ejecutar Migraciones

Una vez que los contenedores estén en ejecución, realiza las migraciones necesarias para preparar la base de datos de Chatwoot:

```bash
docker-compose exec rails bundle exec rails db:chatwoot_prepare
```

- **Explicación**:  
  Este comando ejecuta las migraciones dentro del contenedor `rails`, configurando la base de datos para que Chatwoot funcione correctamente. Si encuentras errores, revisa los logs con:
  ```bash
  docker-compose logs
  ```

### 3. Configurar Claves de Acceso (API Keys)

Para conectar aplicaciones externas (como WhatsApp) con Chatwoot a través de su API, sigue estos pasos:

1. **Accede a la Interfaz de Chatwoot**:  
   Inicia sesión en tu instancia de Chatwoot (por ejemplo, `https://chatwoot.ghlingenieros.com`).

2. **Generar un Access Token**:  
   - Dirígete a **Profile > Settings**.  
   - En la sección **Access Token**, haz clic en generar una nueva clave.  
   - Copia el token generado (este será tu `CHATWOOT_API_KEY`) y guárdalo de forma segura.
  
     ![Image](https://github.com/user-attachments/assets/68d8abb1-441e-4782-a516-822fcdf1f4e0)

3. **Configurar un Inbox para WhatsApp**:  
   - Ve a **Settings > Inboxes**.  
   - Haz clic en **Add Inbox** y selecciona el canal **WhatsApp**.  
   - Ingresa los detalles necesarios, como las credenciales de la API de WhatsApp Business o la integración con un proveedor como Twilio.  
   - Una vez creado, anota el `inbox_id` (visible en la URL o en la configuración del inbox, por ejemplo, `inbox/2`).
  
     ![Image](https://github.com/user-attachments/assets/1cce5e96-4a37-4d4f-b621-1c9e5d22aa8d)

4. **Obtener los Identificadores Necesarios**:  
   - El `account_id` se deriva de la estructura de la URL de la API (por ejemplo, `accounts/1` indica `account_id: 1`).  
   - Combina `account_id` y `inbox_id` para referencias futuras (por ejemplo, `accounts/1/inbox/2`).

5. **Actualizar el Archivo `.env`**:  
   Edita el archivo `.env` en el directorio raíz con las siguientes variables:
   ```bash
   CHATWOOT_API_KEY=tu_access_token_generado
   CHATWOOT_ACCOUNT_ID=1
   INBOX_ID=2
   WHATSAPP_VERIFY_TOKEN=mi_token_secreto
   ```
   Guarda los cambios y reinicia los contenedores:
   ```bash
   docker-compose -f docker-compose.production.yaml restart
   ```

## Verificación

Para confirmar que la integración funciona correctamente:

- Envía un mensaje de prueba desde WhatsApp al número asociado con el inbox configurado.
- Inicia sesión en Chatwoot y navega a **Inboxes > botGhl** (o el nombre del inbox correspondiente a `inbox_id: 2`).
- Verifica que el mensaje aparezca en el hilo de conversación del contacto (por ejemplo, `+51993620749`).

## Solución de Problemas

- **Duplicación de Conversaciones**:  
  Si se crean múltiples conversaciones para el mismo contacto, verifica que el `INBOX_ID` en `.env` coincida con el inbox usado y que las conversaciones no estén marcadas como cerradas en Chatwoot.

## Notas Adicionales

- Consulta la [documentación oficial de la API de Chatwoot](https://www.chatwoot.com/developers/api/) para más detalles sobre los endpoints y parámetros.
- Asegúrate de que el canal WhatsApp esté completamente configurado en Chatwoot para permitir la sincronización de mensajes.
- Para contribuir al proyecto, abre un issue o envía un pull request en el repositorio.

## Configuración de Ejemplo del `.env`

| Variable                | Descripción                              | Valor de Ejemplo       |
|-------------------------|------------------------------------------|------------------------|
| `CHATWOOT_API_KEY`      | Clave API para acceso a Chatwoot         | `tu_access_token`      |
| `CHATWOOT_ACCOUNT_ID`   | ID de la cuenta desde la URL de Chatwoot | `1`                    |
| `INBOX_ID`              | ID del inbox para el canal WhatsApp      | `2`                    |
