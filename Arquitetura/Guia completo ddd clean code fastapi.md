# 🏗️ Guia Completo: DDD + Clean Code com FastAPI

## 📚 Índice

1. [Introdução ao DDD](#introdução-ao-ddd)
2. [Princípios Fundamentais](#princípios-fundamentais)
3. [Arquitetura em Camadas](#arquitetura-em-camadas)
4. [Estrutura do Projeto](#estrutura-do-projeto)
5. [Core - Configurações Centrais](#core---configurações-centrais)
6. [Common - Utilitários Compartilhados](#common---utilitários-compartilhados)
7. [Providers - Abstrações de Terceiros](#providers---abstrações-de-terceiros)
8. [Modules - Lógica de Negócio](#modules---lógica-de-negócio)
9. [Fluxo Completo de uma Requisição](#fluxo-completo-de-uma-requisição)
10. [Testes](#testes)
11. [Boas Práticas e Verdades](#boas-práticas-e-verdades)

---

## 🎯 Introdução ao DDD

### O que é Domain-Driven Design?

DDD **não é sobre pastas e arquivos**. É sobre **organizar o código em torno do negócio**, não da tecnologia.

**Exemplo prático:**
```
❌ ERRADO - Organizado por tecnologia:
/controllers
/services  
/models
/utils

✅ CERTO - Organizado por domínio:
/modules
  /user      ← Tudo sobre usuários
  /auth      ← Tudo sobre autenticação
  /product   ← Tudo sobre produtos
```

### Por que DDD?

1. **Manutenibilidade** - Encontrar código é intuitivo
2. **Escalabilidade** - Novos módulos não afetam existentes
3. **Testabilidade** - Cada módulo testa isoladamente
4. **Clareza** - Código reflete o negócio

---

## 🧩 Princípios Fundamentais

### 1. Separação de Responsabilidades (SRP)

Cada classe tem **UMA** responsabilidade.

```python
# ❌ ERRADO - Classe faz tudo
class UserService:
    def create_user(self, data):
        # Valida
        # Hasheia senha
        # Salva no banco
        # Envia email
        # Gera log
        pass

# ✅ CERTO - Cada classe uma responsabilidade
class CreateUserUseCase:
    def execute(self, data):
        # Apenas orquestra
        pass

class UserRepository:
    def create(self, user):
        # Apenas salva no banco
        pass

class MailProvider:
    def send(self, email):
        # Apenas envia email
        pass
```

### 2. Dependency Inversion (DIP)

**Dependa de abstrações, não de implementações.**

```python
# ❌ ERRADO - Depende de implementação concreta
class CreateUserUseCase:
    def __init__(self):
        self.repo = SQLUserRepository()  # ← Acoplamento!

# ✅ CERTO - Depende de abstração (interface)
class CreateUserUseCase:
    def __init__(self, repo: UserRepository):  # ← Interface!
        self.repo = repo

# Agora pode injetar qualquer implementação:
use_case = CreateUserUseCase(SQLUserRepository())      # SQL
use_case = CreateUserUseCase(MongoUserRepository())    # MongoDB
use_case = CreateUserUseCase(FakeUserRepository())     # Testes
```

### 3. Open/Closed Principle

**Aberto para extensão, fechado para modificação.**

```python
# ✅ Adicionar nova implementação sem modificar código existente
class EmailProvider(ABC):
    @abstractmethod
    def send(self, email): pass

# Implementações (extensões)
class SendGridProvider(EmailProvider): ...
class SMTPProvider(EmailProvider): ...
class AwsSESProvider(EmailProvider): ...  # ← Nova, sem quebrar nada
```

---

## 🏛️ Arquitetura em Camadas

```
┌─────────────────────────────────────────────┐
│         📱 PRESENTATION LAYER               │
│    FastAPI (routes, controllers, schemas)  │
│    - Recebe HTTP                            │
│    - Valida entrada                         │
│    - Retorna JSON                           │
├─────────────────────────────────────────────┤
│         🎯 APPLICATION LAYER                │
│         Use Cases (regras de aplicação)    │
│    - Orquestra fluxo                        │
│    - Regras de negócio                      │
│    - Independente de framework              │
├─────────────────────────────────────────────┤
│         💼 DOMAIN LAYER                     │
│    Entities, Value Objects, Interfaces     │
│    - Regras de negócio puras                │
│    - Sem dependências externas              │
│    - Coração da aplicação                   │
├─────────────────────────────────────────────┤
│         🔧 INFRASTRUCTURE LAYER             │
│    Repositories, Providers, Database        │
│    - Detalhes técnicos                      │
│    - Implementações concretas               │
│    - Acesso a dados, APIs, etc              │
└─────────────────────────────────────────────┘
```

**Regra de ouro:** Camadas superiores podem depender das inferiores, **MAS NUNCA O CONTRÁRIO**.

---

## 📁 Estrutura do Projeto

```
📦 backend/
├── 📁 app/
│   ├── 📁 core/                    ← Configurações centrais
│   │   ├── config.py               ← Settings (env, secrets)
│   │   ├── database.py             ← Conexão DB
│   │   ├── security.py             ← JWT, hash de senha
│   │   └── dependencies.py         ← Dependências compartilhadas
│   │
│   ├── 📁 common/                  ← Utilitários compartilhados
│   │   ├── exceptions.py           ← Exceções customizadas
│   │   ├── responses.py            ← Respostas padronizadas
│   │   └── utils.py                ← Helpers gerais
│   │
│   ├── 📁 providers/               ← Abstrações de terceiros
│   │   └── 📁 mail/
│   │       ├── interfaces/         ← Contratos
│   │       │   └── mail_provider.py
│   │       └── implementations/    ← Implementações concretas
│   │           ├── sendgrid_provider.py
│   │           └── smtp_provider.py
│   │
│   ├── 📁 modules/                 ← Lógica de negócio (DDD)
│   │   └── 📁 user/                ← Bounded Context
│   │       ├── models.py           ← Entidade do banco
│   │       ├── schemas.py          ← DTOs (Pydantic)
│   │       ├── interfaces/         ← Contratos do módulo
│   │       │   └── user_repository.py
│   │       ├── repositories/       ← Implementações de persistência
│   │       │   └── user_repository_impl.py
│   │       ├── use_cases/          ← Casos de uso (regras)
│   │       │   ├── create_user.py
│   │       │   ├── get_user_by_id.py
│   │       │   └── update_user.py
│   │       ├── controllers.py      ← Orquestração HTTP
│   │       ├── routes.py           ← Endpoints FastAPI
│   │       └── tests/              ← Testes do módulo
│   │
│   ├── main.py                     ← Aplicação FastAPI
│   └── routes.py                   ← Agregador de rotas
│
├── .env                            ← Variáveis de ambiente
├── requirements.txt                ← Dependências
└── alembic.ini                     ← Migrations
```

---

## ⚙️ CORE - Configurações Centrais

### `config.py` - Configurações com Pydantic

```python
from pydantic_settings import BaseSettings
from functools import lru_cache

class Settings(BaseSettings):
    """Configurações da aplicação"""
    
    # App
    PROJECT_NAME: str = "My API"
    DEBUG: bool = False
    
    # Database
    DATABASE_URL: str
    
    # JWT
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    
    class Config:
        env_file = ".env"

@lru_cache()  # Singleton
def get_settings() -> Settings:
    return Settings()

settings = get_settings()
```

**Por quê?**
- ✅ Validação automática de tipos
- ✅ Carrega do .env automaticamente
- ✅ Singleton (uma única instância)
- ✅ Type hints para autocomplete

---

### `database.py` - Conexão com Banco

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from typing import Generator
from .config import settings

# Engine
engine = create_engine(settings.DATABASE_URL)

# Session factory
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Base para models
Base = declarative_base()

# Dependency Injection
def get_db() -> Generator[Session, None, None]:
    """Fornece sessão do banco para cada request"""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

**Uso no FastAPI:**
```python
@app.get("/users")
def list_users(db: Session = Depends(get_db)):
    # db já está injetado!
    users = db.query(User).all()
    return users
```

---

### `security.py` - JWT e Hash

```python
from datetime import datetime, timedelta
from jose import jwt
from passlib.context import CryptContext
from .config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    """Gera hash bcrypt da senha"""
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    """Verifica senha contra hash"""
    return pwd_context.verify(plain, hashed)

def create_access_token(data: dict) -> str:
    """Cria JWT token"""
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    
    return jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)

def decode_token(token: str) -> dict:
    """Decodifica e valida JWT"""
    return jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
```

---

### `dependencies.py` - Dependências Reutilizáveis

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer
from .security import decode_token

security = HTTPBearer()

async def get_current_user_id(credentials = Depends(security)) -> str:
    """Extrai user_id do JWT token"""
    try:
        token = credentials.credentials
        payload = decode_token(token)
        user_id = payload.get("sub")
        
        if not user_id:
            raise HTTPException(status_code=401, detail="Token inválido")
        
        return user_id
    except:
        raise HTTPException(status_code=401, detail="Não autorizado")
```

**Uso:**
```python
@app.get("/me")
def get_me(user_id: str = Depends(get_current_user_id)):
    # user_id já validado e extraído!
    return {"user_id": user_id}
```

---

## 🛠️ COMMON - Utilitários Compartilhados

### `exceptions.py` - Exceções de Domínio

```python
class DomainException(Exception):
    """Exceção base do domínio"""
    def __init__(self, message: str, status_code: int = 400):
        self.message = message
        self.status_code = status_code
        super().__init__(self.message)

class EntityNotFoundException(DomainException):
    """Entidade não encontrada"""
    def __init__(self, entity_name: str, entity_id: str):
        message = f"{entity_name} com ID {entity_id} não encontrado"
        super().__init__(message, status_code=404)

class ValidationException(DomainException):
    """Erro de validação de negócio"""
    def __init__(self, message: str):
        super().__init__(message, status_code=422)
```

**Uso:**
```python
# No use case
if not user:
    raise EntityNotFoundException("User", user_id)

if email_exists:
    raise ValidationException("Email já cadastrado")
```

---

### `responses.py` - Respostas Padronizadas

```python
from typing import TypeVar, Generic, Optional
from pydantic import BaseModel

T = TypeVar('T')

class SuccessResponse(BaseModel, Generic[T]):
    """Resposta de sucesso padronizada"""
    success: bool = True
    message: str
    data: Optional[T] = None

def success(message: str, data = None):
    """Helper para resposta de sucesso"""
    return {
        "success": True,
        "message": message,
        "data": data
    }

def error(message: str, errors: dict = None):
    """Helper para resposta de erro"""
    return {
        "success": False,
        "message": message,
        "errors": errors
    }
```

**Uso:**
```python
# Controller
return success("Usuário criado", data=user_dict)
return error("Validação falhou", errors={"email": "Inválido"})
```

---

## 🔌 PROVIDERS - Abstrações de Terceiros

Providers isolam dependências externas (email, storage, pagamento, etc).

### Estrutura

```
providers/mail/
├── interfaces/
│   └── mail_provider.py       ← Interface (contrato)
└── implementations/
    ├── sendgrid_provider.py   ← Implementação SendGrid
    └── smtp_provider.py       ← Implementação SMTP
```

### Interface (Contrato)

```python
# providers/mail/interfaces/mail_provider.py
from abc import ABC, abstractmethod
from typing import List

class MailProvider(ABC):
    """Interface para envio de emails"""
    
    @abstractmethod
    async def send_email(
        self,
        to: List[str],
        subject: str,
        body: str,
        html: bool = False
    ) -> bool:
        """
        Envia email.
        
        Returns:
            True se enviado com sucesso
        """
        pass
```

### Implementação SendGrid

```python
# providers/mail/implementations/sendgrid_provider.py
from sendgrid import SendGridAPIClient
from sendgrid.helpers.mail import Mail
from ..interfaces.mail_provider import MailProvider

class SendGridProvider(MailProvider):
    """Implementação usando SendGrid"""
    
    def __init__(self, api_key: str, from_email: str):
        self.client = SendGridAPIClient(api_key)
        self.from_email = from_email
    
    async def send_email(self, to, subject, body, html=False) -> bool:
        try:
            message = Mail(
                from_email=self.from_email,
                to_emails=to,
                subject=subject,
                html_content=body if html else None
            )
            
            response = self.client.send(message)
            return response.status_code == 202
        except:
            return False
```

### Implementação SMTP

```python
# providers/mail/implementations/smtp_provider.py
import smtplib
from ..interfaces.mail_provider import MailProvider

class SMTPProvider(MailProvider):
    """Implementação usando SMTP"""
    
    def __init__(self, host: str, port: int, user: str, password: str):
        self.host = host
        self.port = port
        self.user = user
        self.password = password
    
    async def send_email(self, to, subject, body, html=False) -> bool:
        # Implementação SMTP...
        pass
```

### Por que usar Providers?

```python
# ✅ Use case depende da INTERFACE, não da implementação
class CreateUserUseCase:
    def __init__(
        self,
        user_repo: UserRepository,
        mail_provider: MailProvider  # ← Interface!
    ):
        self.user_repo = user_repo
        self.mail = mail_provider

# Pode trocar implementação facilmente:
use_case = CreateUserUseCase(
    user_repo,
    SendGridProvider(api_key, from_email)  # Produção
)

use_case = CreateUserUseCase(
    user_repo,
    SMTPProvider(host, port, user, pwd)    # Dev
)

use_case = CreateUserUseCase(
    user_repo,
    FakeMailProvider()                     # Testes
)
```

---

## 📦 MODULES - Lógica de Negócio (DDD)

Cada módulo = 1 Bounded Context (contexto delimitado).

### Anatomia de um Módulo

```
modules/user/
├── models.py              ← Entidade do banco (SQLAlchemy)
├── schemas.py             ← DTOs (Pydantic)
├── interfaces/            ← Contratos
│   └── user_repository.py
├── repositories/          ← Implementações
│   └── user_repository_impl.py
├── use_cases/             ← Regras de negócio
│   ├── create_user.py
│   └── get_user_by_id.py
├── controllers.py         ← Orquestração HTTP → Use Case
└── routes.py              ← Endpoints FastAPI
```

---

### 1. MODELS - Entidade do Banco

```python
# modules/user/models.py
from sqlalchemy import Column, String, Boolean, DateTime
from sqlalchemy.dialects.postgresql import UUID
from datetime import datetime
import uuid
from app.core.database import Base

class User(Base):
    """Entidade User no banco"""
    __tablename__ = "users"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    email = Column(String(255), unique=True, nullable=False, index=True)
    full_name = Column(String(255), nullable=False)
    hashed_password = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

**Características:**
- ✅ Apenas mapeamento ORM
- ✅ Sem lógica de negócio complexa
- ✅ UUID para segurança
- ✅ Timestamps automáticos

---

### 2. SCHEMAS - DTOs (Pydantic)

```python
# modules/user/schemas.py
from pydantic import BaseModel, EmailStr, Field, validator
from datetime import datetime
import uuid

# INPUT SCHEMAS
class UserCreateSchema(BaseModel):
    """Schema para criar usuário"""
    email: EmailStr
    full_name: str = Field(..., min_length=3, max_length=255)
    password: str = Field(..., min_length=8)
    
    @validator('password')
    def validate_password(cls, v):
        if not any(char.isdigit() for char in v):
            raise ValueError('Senha deve conter número')
        return v

class UserUpdateSchema(BaseModel):
    """Schema para atualizar (todos opcionais)"""
    email: EmailStr | None = None
    full_name: str | None = None
    is_active: bool | None = None

# OUTPUT SCHEMAS
class UserResponseSchema(BaseModel):
    """Schema de resposta (sem senha!)"""
    id: uuid.UUID
    email: EmailStr
    full_name: str
    is_active: bool
    created_at: datetime
    
    class Config:
        from_attributes = True  # Permite criar de ORM model
```

**Por quê?**
- ✅ Validação automática na entrada
- ✅ Documentação automática (Swagger)
- ✅ Segurança (não expõe senha)
- ✅ Type hints

---

### 3. INTERFACES - Contratos (Dependency Inversion)

```python
# modules/user/interfaces/user_repository.py
from abc import ABC, abstractmethod
from typing import Optional, List
from ..models import User
from ..schemas import UserCreateSchema, UserUpdateSchema

class UserRepository(ABC):
    """
    Interface do repositório.
    Define O QUE fazer, não COMO.
    """
    
    @abstractmethod
    def create(self, user_data: UserCreateSchema, hashed_password: str) -> User:
        """Cria usuário"""
        pass
    
    @abstractmethod
    def get_by_id(self, user_id: str) -> Optional[User]:
        """Busca por ID"""
        pass
    
    @abstractmethod
    def get_by_email(self, email: str) -> Optional[User]:
        """Busca por email"""
        pass
    
    @abstractmethod
    def update(self, user_id: str, user_data: UserUpdateSchema) -> Optional[User]:
        """Atualiza usuário"""
        pass
    
    @abstractmethod
    def delete(self, user_id: str) -> bool:
        """Deleta (soft delete)"""
        pass
    
    @abstractmethod
    def exists_by_email(self, email: str) -> bool:
        """Verifica se email existe"""
        pass
```

**Benefícios:**
1. **Flexibilidade** - Pode ter SQLRepository, MongoRepository, APIRepository
2. **Testabilidade** - Mock fácil em testes
3. **SOLID** - Dependency Inversion Principle

---

### 4. REPOSITORIES - Implementação de Persistência

```python
# modules/user/repositories/user_repository_impl.py
from sqlalchemy.orm import Session
from ..interfaces.user_repository import UserRepository
from ..models import User
from ..schemas import UserCreateSchema, UserUpdateSchema

class UserRepositoryImpl(UserRepository):
    """Implementação SQL do repositório"""
    
    def __init__(self, db: Session):
        self.db = db
    
    def create(self, user_data: UserCreateSchema, hashed_password: str) -> User:
        user = User(
            email=user_data.email,
            full_name=user_data.full_name,
            hashed_password=hashed_password
        )
        self.db.add(user)
        self.db.commit()
        self.db.refresh(user)
        return user
    
    def get_by_id(self, user_id: str) -> User | None:
        return self.db.query(User).filter(User.id == user_id).first()
    
    def get_by_email(self, email: str) -> User | None:
        return self.db.query(User).filter(User.email == email).first()
    
    def update(self, user_id: str, user_data: UserUpdateSchema) -> User | None:
        user = self.get_by_id(user_id)
        if not user:
            return None
        
        update_data = user_data.model_dump(exclude_unset=True)
        for field, value in update_data.items():
            setattr(user, field, value)
        
        self.db.commit()
        self.db.refresh(user)
        return user
    
    def delete(self, user_id: str) -> bool:
        user = self.get_by_id(user_id)
        if not user:
            return False
        
        user.is_active = False  # Soft delete
        self.db.commit()
        return True
    
    def exists_by_email(self, email: str) -> bool:
        return self.db.query(User).filter(User.email == email).first() is not None
```

**Responsabilidades:**
- ✅ APENAS persistência de dados
- ✅ Sem regras de negócio complexas
- ✅ Implementa TODA a interface

---

### 5. USE CASES - Regras de Negócio

**Use Case = 1 ação do usuário = 1 arquivo**

#### `create_user.py`

```python
# modules/user/use_cases/create_user.py
from ..interfaces.user_repository import UserRepository
from ..schemas import UserCreateSchema, UserResponseSchema
from app.core.security import hash_password
from app.common.exceptions import ValidationException

class CreateUserUseCase:
    """
    Caso de uso: Criar usuário
    
    REGRAS:
    1. Email deve ser único
    2. Senha deve ser hasheada
    3. Usuário inicia ativo
    """
    
    def __init__(self, user_repository: UserRepository):
        self.user_repository = user_repository
    
    async def execute(self, user_data: UserCreateSchema) -> UserResponseSchema:
        # REGRA 1: Email único
        if self.user_repository.exists_by_email(user_data.email):
            raise ValidationException("Email já cadastrado")
        
        # REGRA 2: Hash da senha
        hashed_password = hash_password(user_data.password)
        
        # REGRA 3: Criar ativo
        user = self.user_repository.create(user_data, hashed_password)
        
        return UserResponseSchema.model_validate(user)
```

#### `get_user_by_id.py`

```python
# modules/user/use_cases/get_user_by_id.py
from ..interfaces.user_repository import UserRepository
from ..schemas import UserResponseSchema
from app.common.exceptions import EntityNotFoundException

class GetUserByIdUseCase:
    """Caso de uso: Buscar usuário"""
    
    def __init__(self, user_repository: UserRepository):
        self.user_repository = user_repository
    
    async def execute(self, user_id: str) -> UserResponseSchema:
        user = self.user_repository.get_by_id(user_id)
        
        if not user or not user.is_active:
            raise EntityNotFoundException("User", user_id)
        
        return UserResponseSchema.model_validate(user)
```

#### `update_user.py`

```python
# modules/user/use_cases/update_user.py
from ..interfaces.user_repository import UserRepository
from ..schemas import UserUpdateSchema, UserResponseSchema
from app.common.exceptions import EntityNotFoundException, ValidationException

class UpdateUserUseCase:
    """
    Caso de uso: Atualizar usuário
    
    REGRAS:
    1. Usuário deve existir
    2. Novo email deve ser único
    3. Não pode desativar a si mesmo
    """
    
    def __init__(self, user_repository: UserRepository):
        self.user_repository = user_repository
    
    async def execute(
        self,
        user_id: str,
        user_data: UserUpdateSchema,
        current_user_id: str
    ) -> UserResponseSchema:
        # REGRA 1
        user = self.user_repository.get_by_id(user_id)
        if not user:
            raise EntityNotFoundException("User", user_id)
        
        # REGRA 2
        if user_data.email and user_data.email != user.email:
            if self.user_repository.exists_by_email(user_data.email):
                raise ValidationException("Email já cadastrado")
        
        # REGRA 3
        if user_data.is_active is False and user_id == current_user_id:
            raise ValidationException("Não pode desativar sua própria conta")
        
        updated_user = self.user_repository.update(user_id, user_data)
        return UserResponseSchema.model_validate(updated_user)
```

**Características dos Use Cases:**
- ✅ **Uma responsabilidade** - Um caso de uso
- ✅ **Regras explícitas** - Comentadas e claras
- ✅ **Independente de framework** - Não sabe que é FastAPI
- ✅ **Testável** - Mock do repositório

---

### 6. CONTROLLERS - Orquestração

```python
# modules/user/controllers.py
from fastapi import Depends
from sqlalchemy.orm import Session
from app.core.database import get_db
from app.core.dependencies import get_current_user_id
from app.common.responses import success, error
from app.common.exceptions import DomainException

from .schemas import UserCreateSchema, UserUpdateSchema
from .repositories.user_repository_impl import UserRepositoryImpl
from .use_cases.create_user import CreateUserUseCase
from .use_cases.get_user_by_id import GetUserByIdUseCase
from .use_cases.update_user import UpdateUserUseCase

class UserController:
    """
    Controller de usuários.
    
    RESPONSABILIDADE:
    - Recebe request HTTP
    - Instancia use case com dependências
    - Chama use case
    - Retorna resposta formatada
    """
    
    @staticmethod
    async def create_user(
        user_data: UserCreateSchema,
        db: Session = Depends(get_db)
    ):
        try:
            repo = UserRepositoryImpl(db)
            use_case = CreateUserUseCase(repo)
            user = await use_case.execute(user_data)
            
            return success("Usuário criado com sucesso", data=user.model_dump())
        except DomainException as e:
            return error(message=e.message)
    
    @staticmethod
    async def get_user(
        user_id: str,
        db: Session = Depends(get_db)
    ):
        try:
            repo = UserRepositoryImpl(db)
            use_case = GetUserByIdUseCase(repo)
            user = await use_case.execute(user_id)
            
            return success("Usuário encontrado", data=user.model_dump())
        except DomainException as e:
            return error(message=e.message)
    
    @staticmethod
    async def update_user(
        user_id: str,
        user_data: UserUpdateSchema,
        db: Session = Depends(get_db),
        current_user_id: str = Depends(get_current_user_id)
    ):
        try:
            repo = UserRepositoryImpl(db)
            use_case = UpdateUserUseCase(repo)
            user = await use_case.execute(user_id, user_data, current_user_id)
            
            return success("Usuário atualizado", data=user.model_dump())
        except DomainException as e:
            return error(message=e.message)
```

**Por que separar controller?**
- ✅ Orquestra dependências
- ✅ Pode ser usado por REST, GraphQL, CLI
- ✅ Testes sem HTTP
- ✅ Routes fica limpo

---

### 7. ROUTES - Endpoints FastAPI

```python
# modules/user/routes.py
from fastapi import APIRouter, status
from .controllers import UserController
from .schemas import UserCreateSchema, UserUpdateSchema, UserResponseSchema
from app.common.responses import SuccessResponse

router = APIRouter(prefix="/users", tags=["Users"])

@router.post(
    "",
    response_model=SuccessResponse[UserResponseSchema],
    status_code=status.HTTP_201_CREATED,
    summary="Criar usuário"
)
async def create_user(user_data: UserCreateSchema):
    """
    Cria um novo usuário.
    
    - **email**: Email único
    - **full_name**: Nome completo
    - **password**: Mínimo 8 caracteres com número
    """
    return await UserController.create_user(user_data)

@router.get("/{user_id}", summary="Buscar usuário")
async def get_user(user_id: str):
    return await UserController.get_user(user_id)

@router.patch("/{user_id}", summary="Atualizar usuário")
async def update_user(user_id: str, user_data: UserUpdateSchema):
    return await UserController.update_user(user_id, user_data)
```

---

## 🔄 Fluxo Completo de uma Requisição

```
1. HTTP Request
   POST /users
   {
     "email": "user@example.com",
     "full_name": "João Silva",
     "password": "Senha123"
   }
   ↓

2. routes.py
   - FastAPI recebe
   - Valida schema com Pydantic
   - Chama controller
   ↓

3. controllers.py
   - Injeta dependências (db session)
   - Instancia repositório
   - Instancia use case com repositório
   - Chama use case.execute()
   ↓

4. use_cases/create_user.py
   - Verifica se email já existe (via repository)
   - Hasheia senha (via security)
   - Cria usuário (via repository)
   - Retorna UserResponseSchema
   ↓

5. repositories/user_repository_impl.py
   - Cria entidade User
   - Salva no banco (SQLAlchemy)
   - Retorna User
   ↓

6. controllers.py
   - Recebe UserResponseSchema
   - Formata resposta com success()
   ↓

7. HTTP Response
   {
     "success": true,
     "message": "Usuário criado com sucesso",
     "data": {
       "id": "uuid...",
       "email": "user@example.com",
       "full_name": "João Silva",
       "is_active": true,
       "created_at": "2024-01-01T10:00:00"
     }
   }
```

---

## 🧪 TESTES

### Estrutura de Testes

```
modules/user/tests/
├── test_create_user.py
├── test_get_user.py
└── test_update_user.py
```

### Testando Use Cases (Unidade)

```python
# modules/user/tests/test_create_user.py
import pytest
from app.modules.user.use_cases.create_user import CreateUserUseCase
from app.modules.user.schemas import UserCreateSchema
from app.common.exceptions import ValidationException

class FakeUserRepository:
    """Implementação fake para testes"""
    def __init__(self):
        self.users = {}
    
    def exists_by_email(self, email: str) -> bool:
        return email in [u.email for u in self.users.values()]
    
    def create(self, user_data, hashed_password):
        from app.modules.user.models import User
        import uuid
        
        user = User(
            id=uuid.uuid4(),
            email=user_data.email,
            full_name=user_data.full_name,
            hashed_password=hashed_password
        )
        self.users[user.id] = user
        return user

@pytest.fixture
def user_repository():
    return FakeUserRepository()

@pytest.fixture
def use_case(user_repository):
    return CreateUserUseCase(user_repository)

async def test_create_user_success(use_case):
    """Testa criação bem-sucedida"""
    user_data = UserCreateSchema(
        email="test@example.com",
        full_name="Test User",
        password="Password123"
    )
    
    result = await use_case.execute(user_data)
    
    assert result.email == "test@example.com"
    assert result.full_name == "Test User"
    assert result.is_active == True

async def test_create_user_duplicate_email(use_case, user_repository):
    """Testa erro ao criar com email duplicado"""
    # Cria primeiro usuário
    user_data = UserCreateSchema(
        email="test@example.com",
        full_name="Test User",
        password="Password123"
    )
    await use_case.execute(user_data)
    
    # Tenta criar com mesmo email
    with pytest.raises(ValidationException) as exc:
        await use_case.execute(user_data)
    
    assert "Email já cadastrado" in str(exc.value)
```

### Testando Endpoints (Integração)

```python
# tests/test_user_endpoints.py
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_create_user_endpoint():
    response = client.post("/users", json={
        "email": "test@example.com",
        "full_name": "Test User",
        "password": "Password123"
    })
    
    assert response.status_code == 201
    data = response.json()
    assert data["success"] == True
    assert data["data"]["email"] == "test@example.com"

def test_get_user_endpoint():
    # Primeiro cria
    create_response = client.post("/users", json={
        "email": "test2@example.com",
        "full_name": "Test User 2",
        "password": "Password123"
    })
    user_id = create_response.json()["data"]["id"]
    
    # Depois busca
    response = client.get(f"/users/{user_id}")
    
    assert response.status_code == 200
    data = response.json()
    assert data["data"]["email"] == "test2@example.com"
```

---

## 🎯 BOAS PRÁTICAS E VERDADES

### ✅ O que fazer

1. **Use Cases com UMA responsabilidade**
   ```python
   # ✅ CERTO
   CreateUserUseCase
   UpdateUserEmailUseCase
   DeactivateUserUseCase
   
   # ❌ ERRADO
   UserManagementService  # Faz tudo
   ```

2. **Dependa de interfaces, não implementações**
   ```python
   # ✅ CERTO
   def __init__(self, repo: UserRepository):
   
   # ❌ ERRADO
   def __init__(self, repo: UserRepositoryImpl):
   ```

3. **Schemas separados para input/output**
   ```python
   UserCreateSchema    # Input (com password)
   UserUpdateSchema    # Input parcial
   UserResponseSchema  # Output (sem password)
   ```

4. **Soft delete sempre que possível**
   ```python
   # ✅ CERTO
   user.is_active = False
   
   # ❌ EVITE (perde dados)
   db.delete(user)
   ```

5. **Validações no lugar certo**
   ```python
   Schema (Pydantic)     → Validação de formato
   Use Case              → Validação de negócio
   Repository            → Validação de persistência
   ```

### ❌ O que evitar

1. **Lógica de negócio no controller**
   ```python
   # ❌ ERRADO
   async def create_user(user_data):
       if exists_email(user_data.email):  # ← Regra aqui!
           raise Exception()
   
   # ✅ CERTO
   async def create_user(user_data):
       use_case = CreateUserUseCase(repo)
       return await use_case.execute(user_data)  # ← Regra no use case
   ```

2. **Use case acessar banco diretamente**
   ```python
   # ❌ ERRADO
   class CreateUserUseCase:
       def execute(self, data):
           db.query(User).filter(...)  # ← Acoplamento!
   
   # ✅ CERTO
   class CreateUserUseCase:
       def execute(self, data):
           self.repo.get_by_email(...)  # ← Via interface
   ```

3. **Retornar entidades do banco direto**
   ```python
   # ❌ ERRADO
   return user  # ← User ORM model
   
   # ✅ CERTO
   return UserResponseSchema.model_validate(user)  # ← DTO
   ```

### 💡 Quando usar DDD?

**✅ USE quando:**
- Projeto médio/grande (vai crescer)
- Múltiplos desenvolvedores
- Regras de negócio complexas
- Precisa trocar tecnologias (banco, email, etc)

**❌ NÃO USE quando:**
- Projeto muito simples (CRUD básico)
- Time pequeno, prazo curto
- Prototipagem rápida
- Script one-off

### 🎓 Princípios para lembrar

1. **SOLID**
   - **S**ingle Responsibility
   - **O**pen/Closed
   - **L**iskov Substitution
   - **I**nterface Segregation
   - **D**ependency Inversion

2. **DRY** (Don't Repeat Yourself)
   - Evite duplicação de código
   - Use herança/composição

3. **KISS** (Keep It Simple, Stupid)
   - Simplicidade > Complexidade
   - Não over-engineer

4. **YAGNI** (You Aren't Gonna Need It)
   - Não crie antes de precisar
   - Foco no presente

---

## 📚 Resumo Final

```
┌─────────────────────────────────────────────────────────┐
│                  CAMADAS DA APLICAÇÃO                   │
├─────────────────────────────────────────────────────────┤
│ PRESENTATION (routes.py, controllers.py)                │
│  - FastAPI endpoints                                    │
│  - Validação de entrada (Pydantic)                      │
│  - Formatação de resposta                               │
├─────────────────────────────────────────────────────────┤
│ APPLICATION (use_cases/)                                │
│  - Regras de negócio                                    │
│  - Orquestração de fluxo                                │
│  - Independente de framework                            │
├─────────────────────────────────────────────────────────┤
│ DOMAIN (models.py, schemas.py, interfaces/)             │
│  - Entidades                                            │
│  - Value Objects                                        │
│  - Contratos (interfaces)                               │
├─────────────────────────────────────────────────────────┤
│ INFRASTRUCTURE (repositories/, providers/)              │
│  - Acesso a dados                                       │
│  - Integrações externas                                 │
│  - Implementações concretas                             │
└─────────────────────────────────────────────────────────┘
```

**O fluxo sempre é:**
```
HTTP Request 
  → Route 
    → Controller 
      → Use Case 
        → Repository/Provider 
          → Database/External API
```

**Dependency Inversion:**
```
Use Case depende de → INTERFACE (abstração)
                              ↑
                              |
                      IMPLEMENTAÇÃO concreta
```

---

## 🎯 Checklist do DDD Perfeito

- [ ] Cada use case tem UMA responsabilidade
- [ ] Use cases dependem de interfaces, não implementações
- [ ] Repositórios implementam TODAS as operações da interface
- [ ] Controllers apenas orquestram, sem regras de negócio
- [ ] Schemas separados para input/output
- [ ] Exceções de domínio customizadas
- [ ] Respostas padronizadas (success/error)
- [ ] Providers para abstrair dependências externas
- [ ] Testes unitários dos use cases
- [ ] Testes de integração dos endpoints
- [ ] Documentação automática (FastAPI Swagger)

---

**Versão:** 1.0 - Guia Completo de Estudos
**Autor:** Seu material de estudo de DDD + Clean Code
**Framework:** FastAPI + Python