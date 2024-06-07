# Configuración de Git para Multiples Entornos en MacOS

## Resumen

Este repositorio contiene configuraciones para establecer un entorno Git en MacOS utilizando claves SSH y GPG. Las configuraciones están adaptadas a diferentes directorios de trabajo donde la configuración de Git cambia según el directorio. Esta documentación te guiará a través del proceso de generar y configurar claves SSH y GPG desde cero, y explicará cómo usar los archivos `.gitconfig` y la configuración SSH proporcionados para un entorno de desarrollo fluido.

## Configuración de Claves SSH

### Generar una Nueva Clave SSH

Para generar una nueva clave SSH, abre tu terminal y usa el siguiente comando:

```bash
ssh-keygen -t ed25519 -C "usuario@ejemplo.com"
```

Si utilizas un sistema antiguo que no soporta el algoritmo `Ed25519`, usa:

```bash
ssh-keygen -t rsa -b 4096 -C "usuario@ejemplo.com"
```

Sigue las indicaciones para guardar la clave en la ubicación predeterminada (`~/.ssh/id_ed25519` para claves Ed25519). Cuando se te pida, puedes establecer una frase de contraseña para mayor seguridad.

### Agregar la Clave SSH al Agente SSH

Inicia el agente SSH en segundo plano:

```bash
eval "$(ssh-agent -s)"
```

Agrega tu clave privada SSH al agente SSH:

```bash
ssh-add ~/.ssh/id_ed25519
```

Para sistemas antiguos:

```bash
ssh-add ~/.ssh/id_rsa
```

## Configuración de Claves GPG

### Generar una Nueva Clave GPG

Para generar una nueva clave GPG, usa el siguiente comando:

```bash
gpg --full-generate-key
```

Sigue las indicaciones para configurar tu clave GPG. Elige las siguientes opciones cuando se te pida:
- **Tipo de clave**: RSA y RSA
- **Tamaño de clave**: 4096 bits
- **Expiración de clave**: Elige un período de expiración adecuado o déjalo en "0" para que no expire
- **Nombre real**: Tu Nombre
- **Dirección de correo**: usuario@ejemplo.com
- **Comentario**: Opcional
- **Frase de contraseña**: Establece una frase de contraseña segura

Después de generar la clave, lista tus claves para encontrar el ID de la clave:

```bash
gpg --list-secret-keys --keyid-format LONG
```

Exporta la clave:

```bash
gpg --armor --export tu-id-de-clave
```

## Configuración de Git

### Archivo Principal `.gitconfig`

Crea el archivo principal `.gitconfig` en tu directorio de inicio (`~/.gitconfig`) con el siguiente contenido:

```ini
[user]
  name = Tu Nombre
  email = usuario@dominio.com
  signingkey = tu-cadena-de-clave-ssh-publica

[commit]
  gpgsign = true

[gpg]
  format = ssh

[gpg "ssh"]
  program = /Applications/1Password.app/Contents/MacOS/op-ssh-sign

[includeIf "gitdir:~/Sites/acme/"]
  path = ~/Sites/acme/.gitconfig

[includeIf "gitdir:~/Sites/alpha/"]
  path = ~/Sites/alpha/.gitconfig
```

### Archivos `.gitconfig` Específicos para Directorios

Para el directorio `acme` (`~/Sites/acme/.gitconfig`):

```ini
[user]
  email = usuario@acme.com
  signingkey = tu-cadena-de-clave-gpg

[gpg]
  format = openpgp
```

Para el directorio `alpha` (`~/Sites/alpha/.gitconfig`):

```ini
[user]
  email = usuario@alpha.com
  signingkey = tu-cadena-de-clave-gpg

[gpg]
  format = openpgp
```

### Configuración SSH

Crea un archivo de configuración SSH (`~/.ssh/config`) con el siguiente contenido:

```plaintext
# Uso Personal
Host github.com
    UseKeychain yes
    AddKeysToAgent yes
    IdentityFile ~/.ssh/tu_clave_privada_id_rsa

# Acme
Host github_acme
    HostName github.com
    UseKeychain yes
    AddKeysToAgent yes
    IdentityFile ~/.ssh/tu_clave_privada_acme_id_rsa

# Alpha
Host github_alpha
    HostName github.com
    UseKeychain yes
    AddKeysToAgent yes
    IdentityFile ~/.ssh/tu_clave_privada_alpha_id_rsa
```

## Instrucciones de Uso

1. **Clonar el repositorio**: Usa tu método preferido para clonar el repositorio que contiene esta configuración.

2. **Personalizar `.gitconfig` y configuraciones SSH**: Edita los archivos `.gitconfig` y la configuración SSH con tus propios detalles. Reemplaza los marcadores como `tu-cadena-de-clave-ssh-publica`, `tu-cadena-de-clave-gpg`, y rutas con tu información actual.

3. **Trabajar con diferentes directorios**: La configuración proporcionada aplica automáticamente diferentes configuraciones de Git según el directorio de trabajo (`acme` y `alpha`). Asegúrate de que tu estructura de directorios coincida con la configuración (por ejemplo, `~/Sites/acme` y `~/Sites/alpha`).

4. **Firmar commits**: Tus commits serán firmados utilizando la clave GPG especificada. Asegúrate de agregar tu clave GPG a tu cuenta de GitHub siguiendo la [guía de GitHub](https://docs.github.com/es/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

5. **Uso de claves SSH**: Usa la clave SSH apropiada para diferentes repositorios configurando tus URLs remotas con el host correspondiente (`github_acme` o `github_alpha`).
