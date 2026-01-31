# 🔐 Como um ransomware encripta seus arquivos (sem Hollywood)

> “Ah, mas é só criptografia…”  
> É.  
> **Criptografia bem feita, usada pra ferrar a sua vida.**

Hoje vamos abrir o **WannaCry** no modo *engenharia reversa explicada como gente*, focando **exclusivamente** em uma coisa:
👉 **como ele encripta os arquivos na prática**.

Sem mito.  
Sem glamour hacker.  
Sem tutorial criminoso.  

---

## 🧠 A lógica do ataque (resumo pra quem tem pressa)

Ransomware moderno **não é força bruta**.  
É **criptografia híbrida bem aplicada**.

O WannaCry usa:

- **AES‑128‑CBC** → pra encriptar o conteúdo dos arquivos (rápido)
- **RSA‑2048** → pra proteger a chave do AES (impossível de quebrar)

Tradução:
> O arquivo é trancado com AES.  
> A chave do cadeado é trancada com RSA.  
> Você fica com o arquivo.  
> O atacante fica com a chave.

---

## 🔑 Que chaves existem nesse inferno

### 1️⃣ Chave RSA do atacante (hardcoded)
O malware já vem com uma **RSA Public Key embutida**.
Ela serve pra:
- Proteger tudo que for gerado na vítima
- Garantir que **só o atacante consiga reverter**

Essa chave **não muda por vítima**.

---

### 2️⃣ Chaves geradas na máquina da vítima
Durante a infecção, o WannaCry:
- Gera um **par RSA‑2048 local**
- Salva arquivos como `.pky` e `.eky`
- **Criptografa a chave privada local com a RSA do atacante**

Ou seja:
> Até a chave que nasce na sua máquina… **não é sua**.

---

## 🧨 Agora o que importa: como um arquivo é encriptado

### 🔹 Passo 1 - Uma chave AES por arquivo
Cada arquivo recebe:
- **Uma chave AES‑128 aleatória**
- Gerada via API criptográfica do Windows

Por quê isso é cruel?
> Recuperar uma chave não salva o resto.

---

### 🔹 Passo 2 — AES‑128‑CBC no conteúdo
O arquivo vira ciphertext usando:
- AES
- Modo CBC
- Em várias análises: **IV nulo**

CBC em português:
> Um bloco depende do outro.  
> Quebrou um byte?  
> Já era.

---

### 🔹 Passo 3 - RSA protegendo a chave AES
A chave AES do arquivo:
- É encriptada com **RSA‑2048**
- Usando a chave pública do atacante

Sem a **RSA Private Key correta**:
> Você olha pro arquivo.  
> O arquivo olha pra você.  
> E ninguém colabora.

---

### 🔹 Passo 4 - Arquivo final
O arquivo encriptado contém:
- Marcador (`WANACRY!`)
- Metadados
- Chave AES encriptada
- Conteúdo encriptado
- Extensão alterada (`.WNCRY`)

Backup começa a fazer sentido aqui.

---

## 🧩 Fluxo técnico em uma linha

```
Arquivo → AES‑128‑CBC → Arquivo criptografado
        → RSA‑2048 protege a chave AES
        → Sem chave privada = sem choro
```

---

## 🧪 Mini‑exemplo didático (AES + RSA)

> ⚠️ Exemplo educacional  
> ⚠️ Não é ransomware  
> ⚠️ Só demonstra criptografia híbrida

```python
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

rsa_private = rsa.generate_private_key(public_exponent=65537, key_size=2048)
rsa_public = rsa_private.public_key()

data = b"Backup? Nunca ouvi falar."
aes_key = os.urandom(16)
iv = b"\x00" * 16

cipher = Cipher(algorithms.AES(aes_key), modes.CBC(iv))
encryptor = cipher.encryptor()

pad = 16 - len(data) % 16
data += bytes([pad]) * pad

ciphertext = encryptor.update(data) + encryptor.finalize()

encrypted_key = rsa_public.encrypt(
    aes_key,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None
    )
)
```

---

## 🛡️ Pra defesa

- Não é vírus simples
- Não é senha fraca
- É **perda de material criptográfico**

Defesa real envolve:
- Backup offline
- Detecção comportamental
- EDR antes da fase de crypto
- Controle de escrita e execução
- Plataforma neutralizadora de Ransomware

---

## 📚 Referências técnicas (fontes abertas)

- https://cloud.google.com/blog/topics/threat-intelligence/wannacry-malware-profile  
- https://www.secureworks.com/research/wcry-ransomware-analysis  
- https://serhack.me/articles/technical-analysis-ransomware-wannacry/  
- https://www.malwarebytes.com/blog/news/2017/05/the-wannacry-ransomware-attack  

---

🧠 Criptografia não é vilã.  
☠️ Vilão é quem segura a chave.
