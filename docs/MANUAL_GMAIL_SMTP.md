# Manual de Configuración: Gmail SMTP Gratuito para StetikShop

Gmail permite enviar correos electrónicos de forma **100% gratuita** utilizando su servidor SMTP oficial (`smtp.gmail.com`). 

> [!IMPORTANT]
> Google **no permite** usar tu contraseña habitual de inicio de sesión por motivos de seguridad. Se requiere generar una **"Contraseña de aplicación"** de 16 letras, la cual es gratuita y toma menos de 2 minutos.

---

## Paso 1: Activar la Verificación en Dos Pasos (2FA) en Google

1. Ve a tu cuenta de Google: [https://myaccount.google.com/](https://myaccount.google.com/).
2. En el menú de la izquierda, selecciona **Seguridad**.
3. En la sección *"Cómo inicias sesión en Google"*, asegúrate de tener activada la **Verificación en dos pasos**.
   *(Si ya la tienes activada, puedes pasar directamente al Paso 2).*

---

## Paso 2: Generar la Contraseña de Aplicación (16 letras)

1. Ve directamente al enlace de contraseñas de aplicaciones de Google:  
   👉 [https://myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)
2. En el campo **"Nombre de la aplicación"**, escribe: `StetikShop`.
3. Haz clic en el botón **Crear**.
4. Google te mostrará una ventana con una contraseña de **16 letras amarillas** (ejemplo: `abcd efgh ijkl mnop`).
5. **Copia esa contraseña de 16 letras**.

---

## Paso 3: Pegar tus Credenciales en el Proyecto

Abre el archivo [`server/.env`](file:///c:/src/StetikShop/server/.env) y reemplaza los valores de prueba por tu correo y tu contraseña de 16 letras:

```env
# Credenciales SMTP de Gmail (Gratis)
GMAIL_SMTP_USER=tu-correo-real@gmail.com
GMAIL_SMTP_PASS=abcdefghijklmnop
```

---

## Paso 4: Habilitar el Envíos Externos en Supabase Local

Abre el archivo [`supabase/config.toml`](file:///c:/src/StetikShop/supabase/config.toml) y cambia `enabled = false` a `enabled = true`:

```toml
[auth.email.smtp]
enabled = true
host = "smtp.gmail.com"
port = 587
user = "env(GMAIL_SMTP_USER)"
pass = "env(GMAIL_SMTP_PASS)"
admin_email = "env(GMAIL_SMTP_USER)"
sender_name = "StetikShop"
```

Si deseas que sea **obligatorio confirmar el correo** al registrarse para recibir el enlace en la bandeja de entrada real:
En la línea 203 de [`supabase/config.toml`](file:///c:/src/StetikShop/supabase/config.toml):
```toml
enable_confirmations = true
```

---

## Paso 5: Reiniciar Supabase

Una vez colocados tus valores, ejecuta en tu terminal:

```bash
npx supabase stop
npx supabase start
```

¡Listo! A partir de ese momento, cualquier registro o solicitud de recuperación de contraseña enviará un correo real y directo desde tu cuenta de Gmail a la bandeja de entrada del usuario.
