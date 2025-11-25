---

# Demo Interactiva: Seguridad en Sistemas Distribuidos

## Conceptos Implementados

### 1. Autenticación con Contraseña

### 2. Cifrado/Descifrado Simétrico

### 3. Intercambio de Claves Diffie-Hellman

### 4. Handshake SSL/TLS

---

## 1. CIFRADO SIMÉTRICO

### ¿Qué tipo de cifrado usa?

**NO es cifrado real** - Es una **simulación educativa** usando Base64.

```javascript
const simpleEncrypt = (text, key) => {
    return btoa(text + ':' + key);
};
```

### ¿Por qué NO es cifrado real?

1. Base64 NO es cifrado, es solo codificación
2. Cualquiera puede decodificarlo con `atob()`
3. La "clave" se incluye en el mensaje (muy inseguro)

### ¿Cómo funciona paso a paso?

```javascript
// Usuario escribe: "Hola Mundo"
// Clave hardcodeada: "shared-secret-key"

// PASO 1: Concatenar
mensaje = "Hola Mundo" + ":" + "shared-secret-key"
// Resultado: "Hola Mundo:shared-secret-key"

// PASO 2: Codificar en Base64
cifrado = btoa("Hola Mundo:shared-secret-key")
// Resultado: "SG9sYSBNdW5kbzpzaGFyZWQtc2VjcmV0LWtleQ=="

// PASO 3: Mostrar en pantalla
```

### Para Descifrar:

```javascript
const simpleDecrypt = (encrypted, key) => {
    decoded = atob("SG9sYSBNdW5kbzpzaGFyZWQtc2VjcmV0LWtleQ==")
    
    [texto, claveGuardada] = decoded.split(':')

    if (claveGuardada === "shared-secret-key") {
        return texto;
    } else {
        return null;
    }
};
```

### ¿Por qué usar una simulación?

En producción deberías usar:

* AES-256
* Web Crypto API
* Bibliotecas como CryptoJS

**Ejemplo con cifrado REAL:**

```javascript
const crypto = require('crypto');

function encryptAES(text, key) {
    const cipher = crypto.createCipher('aes-256-cbc', key);
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    return encrypted;
}
```

---

## 2. AUTENTICACIÓN CON CONTRASEÑA

### ¿Cómo funciona?

```javascript
const handlePasswordAuth = () => {
    const correctPassword = 'SecurePass123';
    
    if (password === correctPassword) {
        setAuthStatus('success');
    } else {
        setAuthStatus('failed');
    }
    
    setTimeout(() => setAuthStatus(null), 3000);
};
```

### Flujo Paso a Paso:

```
┌─────────────┐                    ┌─────────────┐
│   USUARIO   │                    │   SISTEMA   │
└──────┬──────┘                    └──────┬──────┘
       │                                  │
       │  1. Ingresa: "SecurePass123"     │
       ├─────────────────────────────────>│
       │                                  │
       │         2. Compara con BD        │
       │            correctPassword        │
       │                                  │
       │  3. Autenticación exitosa        │
       │<─────────────────────────────────┤
       │                                  │
```

### Problemas de Seguridad (Solo Demo):

1. Contraseña en texto plano
2. Sin cifrado en tránsito
3. Sin protección contra fuerza bruta
4. Sin salt

### Implementación Segura:

```javascript
const bcrypt = require('bcrypt');

const saltRounds = 10;
const hashedPassword = await bcrypt.hash('SecurePass123', saltRounds);

const match = await bcrypt.compare(passwordIngresado, hashedPassword);
```

---

## 3. DIFFIE-HELLMAN: INTERCAMBIO DE CLAVES

### ¿Qué problema resuelve?

¿Cómo crear una clave secreta compartida por un canal público?

### Valores Utilizados:

```javascript
const simulateDH = () => {
    const g = 5;
    const n = 23;

    const aliceSecret = 6;
    const bobSecret = 8;

    const alicePublic = Math.pow(g, aliceSecret) % n;
    const bobPublic = Math.pow(g, bobSecret) % n;

    const aliceShared = Math.pow(bobPublic, aliceSecret) % n;
    const bobShared = Math.pow(alicePublic, bobSecret) % n;
};
```

### Ejemplo:

**Alice:**

```
alicePublic = 5^6 mod 23 = 8
```

**Bob:**

```
bobPublic = 5^8 mod 23 = 16
```

**Clave compartida:**

```
Alice: 16^6 mod 23 = 9
Bob:   8^8 mod 23 = 9
```

### Resultado:

```
Ambos obtienen la misma clave: 9
```

### ¿Por qué es seguro?

Porque recuperar el exponente privado (logaritmo discreto) es computacionalmente inviable para números grandes.

---

## 4. HANDSHAKE SSL/TLS

### Secuencia Completa:

```
CLIENTE                                     SERVIDOR
   │                                            │
   │  1. Client Hello                           │
   ├──────────────────────────────────────────>│
   │                                            │
   │                    2. Server Hello         │
   │                    + Certificado           │
   │<──────────────────────────────────────────┤
   │                                            │
   │  3. Verificación del Certificado           │
   │                                            │
   │  4. Key Exchange                           │
   ├──────────────────────────────────────────>│
   │                                            │
   │                    5. Ambos calculan clave │
   │                                            │
   │  6. Finished                               │
   ├──────────────────────────────────────────>│
   │<──────────────────────────────────────────┤
```

### ¿Qué se consigue?

1. Autenticación
2. Confidencialidad
3. Integridad
4. Forward Secrecy

---

## Arquitectura del Código

### Estado de React:

```javascript
const [activeTab, setActiveTab] = useState('authentication');
const [message, setMessage] = useState('');
const [encryptedMessage, setEncryptedMessage] = useState('');
const [decryptedMessage, setDecryptedMessage] = useState('');
const [password, setPassword] = useState('');
const [authStatus, setAuthStatus] = useState(null);
const [sslStep, setSslStep] = useState(0);
const [dhValues, setDhValues] = useState(null);
```

---

## Advertencias Importantes

### NO usar en producción:

1. Cifrado simulado
2. Contraseñas sin hash
3. Sin HTTPS
4. DH con números pequeños
5. Sin manejo de errores

### Para producción usar:

* Web Crypto API
* bcrypt/Argon2
* TLS 1.3
* OpenSSL, libsodium
* Certificate pinning

---

## Recursos Adicionales

* OWASP Cryptography Guide
* TLS 1.3 RFC
* Stanford Crypto Course
* Let's Encrypt
* OpenSSL
* Node.js Crypto

---

## Autor

Creado con fines educativos para visualizar conceptos de seguridad en sistemas distribuidos por Juan Camilo Avendano Rodriguez.

---

## Créditos

Basado en el documento "Distributed System Security" por Peter Reiher (UCLA).

---
