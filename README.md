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

- **AES-128-CBC** → pra encriptar o conteúdo dos arquivos (rápido, feito pra volume)  
- **RSA-2048** → pra proteger a chave do AES (cripto assimétrica, inviável de quebrar)

Tradução humana:
> O arquivo é trancado com AES.  
> A chave do cadeado é trancada com RSA.  
> Você fica com o arquivo.  
> O atacante fica com a chave.

---

## 🔑 Que chaves existem nesse inferno

### 1️⃣ Chave RSA do atacante (hardcoded)
O malware já vem com uma **RSA Public Key embutida no binário**.

Ela serve pra:
- Proteger qualquer material criptográfico gerado na vítima  
- Garantir que **só o operador do ransomware consiga reverter**

📌 Essa chave **não muda por vítima**.

---

### 2️⃣ Chaves geradas na máquina da vítima
Durante a infecção, o WannaCry:
- Gera um **par RSA-2048 local**
- Salva artefatos como `.pky` e `.eky`
- **Criptografa a chave privada local com a RSA do atacante**

Ou seja:
> A chave nasce na sua máquina…  
> mas já nasce **sequestrada**.

---

## 🧨 Como um arquivo é encriptado (sem mistério)

### 🔹 Passo 1 — Uma chave AES por arquivo
Para **cada arquivo**, o ransomware gera:
- **Uma chave AES-128 aleatória**
- Usando APIs criptográficas do sistema

> Uma chave por arquivo impede recuperação em massa.

---

### 🔹 Passo 2 — AES-128-CBC no conteúdo
O conteúdo vira ciphertext usando:
- AES  
- Modo CBC  
- Em várias análises: **IV nulo**

CBC em português:
> Um bloco depende do anterior.  
> Errou um byte?  
> Já era.

---

### 🔹 Passo 3 — RSA protegendo a chave AES
A chave AES:
- É encriptada com **RSA-2048**
- Usando a chave pública do atacante

Sem a chave privada correta:
> Você tem o arquivo.  
> Mas não tem a chave.

---

### 🔹 Passo 4 — Arquivo final
O arquivo resultante contém:
- Marcador (`WANACRY!`)  
- Metadados  
- Chave AES encriptada  
- Conteúdo encriptado  
- Extensão alterada (`.WNCRY`)

Backup começa a fazer sentido aqui.

---

## 🧩 Fluxo técnico resumido

```
Arquivo original
   ↓
AES-128-CBC (chave única por arquivo)
   ↓
Arquivo criptografado
   ↓
Chave AES protegida com RSA-2048
   ↓
Sem chave privada = sem choro
```

---

## 🧪 Exemplo didático (AES + RSA explicado)

> ⚠️ Exemplo educacional  
> ⚠️ Não é ransomware  
> ⚠️ Demonstra apenas criptografia híbrida

```python
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os
```

### 🔹 Gerando o cofre RSA (simulando o atacante)
```python
rsa_private = rsa.generate_private_key(
    public_exponent=65537,
    key_size=2048
)
rsa_public = rsa_private.public_key()
```

### 🔹 Conteúdo do arquivo
```python
data = b"Backup? Nunca ouvi falar."
```

### 🔹 Chave AES e IV
```python
aes_key = os.urandom(16)  # 128 bits
iv = b"\x00" * 16        # IV nulo
```

### 🔹 Encriptação AES
```python
cipher = Cipher(algorithms.AES(aes_key), modes.CBC(iv))
encryptor = cipher.encryptor()

pad = 16 - len(data) % 16
data += bytes([pad]) * pad

ciphertext = encryptor.update(data) + encryptor.finalize()
```

### 🔹 Protegendo a chave AES com RSA
```python
encrypted_key = rsa_public.encrypt(
    aes_key,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None
    )
)
```

Aqui acontece o sequestro real da chave.

---

## 🛡️ Pra defesa

- Não é vírus simples  
- Não é senha fraca  
- É **perda de material criptográfico**

Defesa real envolve:
- Backup offline e testado  
- Detecção comportamental  
- EDR antes da fase de crypto  
- Controle de escrita e execução  

---

## 📚 Referências técnicas

- https://cloud.google.com/blog/topics/threat-intelligence/wannacry-malware-profile  
- https://www.secureworks.com/research/wcry-ransomware-analysis  
- https://serhack.me/articles/technical-analysis-ransomware-wannacry/  
- https://www.malwarebytes.com/blog/news/2017/05/the-wannacry-ransomware-attack  

---

🧠 Criptografia não é vilã.  
☠️ Vilão é quem segura a chave.
