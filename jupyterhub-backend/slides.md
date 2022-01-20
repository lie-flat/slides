---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
fonts:
  mono: 'Cascadia Code'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# persist drawings in exports and build
drawings:
  persist: false
---

## CFPS JupyterHub 大作业讲解

# 后端

#### 山东大学（威海） 2020级 数据科学与人工智能实验班

#### 讲解人：任鹏飞

#### 2022 年 1 月 20 日

幻灯片在 https://github.com/lie-flat/slides 仓库

---

# 目录结构

```
cfps-jupyterhub/backend
├── config.py
├── db.py
├── __init__.py
├── limiter.py
├── schemas.py
├── security.py
└── user.py
```

```
cfps-jupyterhub/server.py
cfps-jupyterhub/users.sqlite
```


---

# 入口脚本
../server.py

```python
import uvicorn
from backend.config import port, root_path, APP_PROFILE

if __name__ == "__main__":
    match APP_PROFILE:
        case "dev":
            uvicorn.run("backend:app", host="127.0.0.1", port=port, reload=True, root_path=root_path)
        case "prod":
            uvicorn.run("backend:app", host="127.0.0.1", port=port, reload=False, root_path=root_path)
```

- 根据配置，以开发模式或生产模式启动服务

---

<div style="float:left;">

# \_\_init\_\_.py
\_\_init\_\_.py

```python
from fastapi import FastAPI
from slowapi import _rate_limit_exceeded_handler
from slowapi.errors import RateLimitExceeded
from slowapi.middleware import SlowAPIMiddleware

import backend.user
from .db import engine, BaseDBModel
from .limiter import limiter

BaseDBModel.metadata.create_all(bind=engine)

app = FastAPI()
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, 
    _rate_limit_exceeded_handler)
app.add_middleware(SlowAPIMiddleware)

@app.get("/")
async def root():
    return "Hello World!"

app.include_router(user.router)
```

</div>

<div style="float: left;margin-left: 20px;">

# config.py

```python
import os
port = 5432
root_path = '/api'
INVITE_CODES = os.environ.get('INVITE_CODES', '123456').split(';')
RATE_LIMIT = os.environ.get("API_RATE_LIMIT", "30/minute")
SECRET_KEY = os.environ.get("API_SECRET_KEY", "*********")
HUB_CLIENT_ID = os.environ.get(
    "API_HUB_CLIENT_ID", "EXAMPLE-HUB-CLIENT-ID")
HUB_CLIENT_SECRET = os.environ.get(
    "API_HUB_CLIENT_SECRET", "SUPER-secret-JUPYTERHUB")
APP_PROFILE = os.environ.get("APP_PROFILE", "dev")
DOMAIN_NAME = os.environ.get("APP_DOMAIN_NAME", "localhost")
```
# limiter.py
```python
from slowapi import Limiter
from slowapi.util import get_remote_address
from .config import RATE_LIMIT
limiter = Limiter(
    key_func=get_remote_address, 
    default_limits=[RATE_LIMIT]
)
```

</div>

---


<div style="float:left;">

# user.py

```python
router = APIRouter()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="user-login")
```

```python
def authenticate_user(db, username, password):
    user = get_user_by_username(db, username)
    if not user:
        return None
    if not verify_password(password, user.hashed_password):
        return None
    return user
```

```python
@router.get("/user-data")
async def user_data(
    current_user: DBUser = Depends(get_current_user)
):
    return \
        { "username": current_user.username,
          "is_active": current_user.is_active }
```

</div> 

<div style="float: left; width: 430px; margin-left: 30px;margin-right: -70px;">

```python
async def get_current_user(
        token: str = Depends(oauth2_scheme), 
        db: Session = Depends(get_db)
):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Invalid authentication credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = decode_token(token)
        username = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except JWTError:
        raise credentials_exception
    user = get_user_by_username(db, 
                username=token_data.username)
    if user is None:
        raise credentials_exception
    return user
```

</div>

<div style="clear: both;">

```python
async def get_current_active_user(current_user: User = Depends(get_current_user)):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="账户已停用")
    return current_user
```
</div>

---

# user.py
Login

```python
@router.post("/user-login")
async def login(form_data: OAuth2PasswordRequestForm = Depends(), db: Session = Depends(get_db)):
    if form_data.client_id == HUB_CLIENT_ID and form_data.client_secret != HUB_CLIENT_SECRET:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid client credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )

    user = authenticate_user(db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="用户名或密码错误",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token = create_access_token({"sub": user.username})
    return {"access_token": access_token, "token_type": "bearer"}
```

---

# user.py
Register

```python
def validate_username(username: str):
    if re.match(r'^[a-zA-Z0-9_]{4,16}$', username):
        return True
    return False

@router.post("/user-register/", response_model=User)
async def register(form_data: RegisterInfo, db: Session = Depends(get_db)):
    if form_data.invite_code not in INVITE_CODES:
        raise HTTPException(status_code=status.HTTP_418_IM_A_TEAPOT, detail="邀请码无效")
    if not validate_username(form_data.username):
        raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail="用户名无效")
    if not validate_password_strength(form_data.password):
        raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail="你的密码太弱了")
    try:
        user = create_user(db, form_data)
    except:
        raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail="用户已存在")
    return User(username=user.username)
```

---

# user.py
/token 

```python
@router.post("/token")
async def token(data: Request):
    body = await data.body()
    body = b"http://nothing?" + body  # Make the URL Parser happy
    qs = parse_qs(urlparse(body).query)
    code = qs.get(b"code", [None])
    if code[0] is not None:
        code = code[0].decode("utf-8") if len(code) > 0 else None
    else:
        raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail="Missing code")
    if not validate_token(code):
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid token")
    return {"access_token": code, "token_type": "bearer"}
```

<div>

<div style="float:left;">

# schemas.py

```python
class User(BaseModel):
    username: str
    disabled: bool | None = False
```

</div>

<div style="float:left; margin-left: 20px;">

```python
from pydantic import BaseModel

class RegisterInfo(BaseModel):
    username: str
    password: str
    invite_code: str
```

</div>
<div style="float:left; margin-left: 20px;">

```python
class Token(BaseModel):
    access_token: str
    token_type: str
```

</div>
<div style="float:left; margin-left: 20px;">

```python
class TokenData(BaseModel):
    username: str | None = None
```

</div>

</div>

---

<div style="float:left;">

# db.py

```python
from sqlalchemy import create_engine, Column, Integer, String, Boolean
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from .schemas import RegisterInfo
from .security import get_password_hash
SQLALCHEMY_DATABASE_URL = "sqlite:///./users.sqlite"
engine = create_engine(
    SQLALCHEMY_DATABASE_URL, 
    connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
BaseDBModel = declarative_base()
class DBUser(BaseDBModel):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)
def get_user(db, uid):
    return db.query(DBUser).filter(DBUser.id == uid).first()
def get_user_by_username(db, username):
    return db.query(DBUser).filter(DBUser.username == username).first()
```

</div>

<div style="float: left; width: 300px; margin-left: 30px;margin-right: -30px;">

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

```python
def create_user(
    db, 
    reg_info: RegisterInfo
):
    hashed = get_password_hash(
        reg_info.password)
    user = DBUser(
        username=reg_info.username, 
        hashed_password=hashed
    )
    db.add(user)
    db.commit()
    db.refresh(user)
    return user
```

</div>

---

# security.py


<div style="float: right">

```python
from datetime import datetime, timedelta
import re
from jose import jwt
from passlib.context import CryptContext
from .config import SECRET_KEY

ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30
pwd_context = CryptContext(
    schemes=["bcrypt"], 
    deprecated="auto"
)

def validate_token(token):
    try:
        decode_token(token)
    except:
        return False
    return True

def get_password_hash(password):
    return pwd_context.hash(password)
```

</div>

```python
def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)
def decode_token(token):
    return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt
def validate_password_strength(password):
    """
    Minimum eight characters, at least one uppercase letter,
    one lowercase letter, one number and one special character
    """
    if not re.match(
        r"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[#?!@$%^&*_=|+-]).{8,128}$", 
        password
    ):
        return False
    return True
```
