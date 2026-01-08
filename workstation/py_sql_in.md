太好了！用Python学习SQL注入和Web安全更直接，因为Python在现代Web开发中应用广泛。我会为你设计**纯Python的渐进式安全项目**，使用主流的Flask框架和SQLAlchemy。

## 🎯 Python安全学习路线图

```
基础项目(1-3) → 进阶项目(4-5) → 综合项目(6)
    ↓            ↓            ↓
SQLite基础    Flask-SQLAlchemy  完整Web应用
简单注入       ORM安全          实战防护
```

---

## 环境准备：一次性配置

### 1. 安装依赖
```bash
# 创建项目目录
mkdir python-sql-security
cd python-sql-security

# 创建虚拟环境（推荐）
python -m venv venv

# Windows激活
venv\Scripts\activate
# Mac/Linux激活
source venv/bin/activate

# 安装所有依赖
pip install flask flask-sqlalchemy flask-wtf bcrypt jinja2
```

### 2. 初始化数据库脚本
创建 `init_database.py`：
```python
import sqlite3
import bcrypt

# 创建数据库和表
conn = sqlite3.connect('vulnerable_app.db')
cursor = conn.cursor()

# 用户表
cursor.execute('''
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    email TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
''')

# 产品表
cursor.execute('''
CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    price REAL NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
''')

# 评论表
cursor.execute('''
CREATE TABLE IF NOT EXISTS comments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users (id)
)
''')

# 插入测试数据
# 1. 用户数据（密码都是：password123）
hashed_password = bcrypt.hashpw('password123'.encode(), bcrypt.gensalt()).decode()

cursor.execute("INSERT OR IGNORE INTO users (username, email, password_hash) VALUES (?, ?, ?)", 
               ('alice', 'alice@example.com', hashed_password))
cursor.execute("INSERT OR IGNORE INTO users (username, email, password_hash) VALUES (?, ?, ?)", 
               ('bob', 'bob@example.com', hashed_password))
cursor.execute("INSERT OR IGNORE INTO users (username, email, password_hash) VALUES (?, ?, ?)", 
               ('charlie', 'charlie@example.com', hashed_password))

# 2. 产品数据
products = [
    ('笔记本电脑', 5999.99, '高性能游戏本'),
    ('智能手机', 2999.50, '最新款智能手机'),
    ('平板电脑', 1999.00, '轻薄便携'),
    ('智能手表', 899.00, '健康监测'),
]
cursor.executemany("INSERT OR IGNORE INTO products (name, price, description) VALUES (?, ?, ?)", products)

conn.commit()
conn.close()

print("✅ 数据库初始化完成！")
print("📊 已创建：users, products, comments 表")
print("👤 测试用户：alice / bob / charlie，密码都是：password123")
```

运行一次：`python init_database.py`

---

## 项目1：最基本的SQL注入演示
**目标**：理解SQL注入的核心原理

### `01_basic_injection.py`
```python
import sqlite3
import html

# 连接到数据库
conn = sqlite3.connect('vulnerable_app.db')
cursor = conn.cursor()

def vulnerable_login(username, password):
    """危险的登录函数 - 直接拼接SQL"""
    # 🚨 危险！直接拼接用户输入
    sql = f"SELECT * FROM users WHERE username = '{username}' AND password_hash = '{password}'"
    print(f"🔴 执行的SQL: {sql}")
    
    try:
        cursor.execute(sql)
        user = cursor.fetchone()
        return user is not None
    except Exception as e:
        print(f"❌ 错误: {e}")
        return False

def simulate_attacks():
    """模拟各种SQL注入攻击"""
    print("=" * 60)
    print("🧪 SQL注入攻击演示")
    print("=" * 60)
    
    # 测试数据
    test_cases = [
        ("正常登录", "alice", "password123", False),
        ("经典注入", "admin' --", "anything", True),  # 绕过密码验证
        ("永真条件", "' OR '1'='1' --", "anything", True),
        ("联合查询", "' UNION SELECT 1,2,3,4 --", "anything", False),
    ]
    
    for name, username, password, expected_bypass in test_cases:
        print(f"\n📌 测试: {name}")
        print(f"   用户名: {username}")
        print(f"   密码: {password}")
        
        result = vulnerable_login(username, password)
        status = "✅ 登录成功（被绕过！）" if result else "❌ 登录失败"
        
        if result == expected_bypass:
            print(f"   结果: {status} [符合预期]")
        else:
            print(f"   结果: {status} [不符合预期]")

def safe_login(username, password):
    """安全的登录函数 - 使用参数化查询"""
    # 🟢 安全：使用参数化查询
    sql = "SELECT * FROM users WHERE username = ? AND password_hash = ?"
    print(f"\n🟢 安全SQL: {sql}")
    print(f"   参数: username={username}, password={password}")
    
    cursor.execute(sql, (username, password))
    user = cursor.fetchone()
    return user is not None

def demonstrate_safe_version():
    """演示安全版本如何防御"""
    print("\n" + "=" * 60)
    print("🛡️  安全版本演示")
    print("=" * 60)
    
    # 同样的攻击，在安全版本中会失败
    attacks = [
        ("admin' --", "anything"),
        ("' OR '1'='1' --", "anything"),
    ]
    
    for username, password in attacks:
        print(f"\n尝试攻击: {username}")
        result = safe_login(username, password)
        print(f"结果: {'❌ 登录失败（安全！）' if not result else '⚠️ 警告：应该失败！'}")

if __name__ == "__main__":
    print("🔐 SQL注入学习项目 - 基础演示")
    print("-" * 60)
    
    # 第一部分：漏洞演示
    simulate_attacks()
    
    # 第二部分：安全演示
    demonstrate_safe_version()
    
    # 清理
    cursor.close()
    conn.close()
```

**运行与学习**：
```bash
python 01_basic_injection.py
```

**学习任务**：
1. 运行代码，观察每种攻击的SQL语句
2. 理解`--`注释符的作用
3. 对比危险版本和安全版本的区别
4. 尝试添加自己的攻击payload

---

## 项目2：Flask Web应用漏洞演示
**目标**：在真实Web环境中体验SQL注入

### `02_flask_vulnerable_app.py`
```python
from flask import Flask, request, render_template_string, render_template
import sqlite3
import os

app = Flask(__name__)
DATABASE = 'vulnerable_app.db'

def get_db_connection():
    """获取数据库连接"""
    conn = sqlite3.connect(DATABASE)
    conn.row_factory = sqlite3.Row  # 返回字典形式的结果
    return conn

# 简单的HTML模板
HTML_TEMPLATE = '''
<!DOCTYPE html>
<html>
<head>
    <title>漏洞商品搜索系统</title>
    <style>
        body { font-family: Arial; max-width: 800px; margin: 40px auto; }
        .vulnerable { background: #ffe6e6; padding: 20px; border-left: 4px solid red; }
        .safe { background: #e6ffe6; padding: 20px; border-left: 4px solid green; }
        input, button { padding: 10px; margin: 5px; }
        .product { border: 1px solid #ccc; padding: 15px; margin: 10px 0; }
        .error { color: red; }
        .sql { background: #f5f5f5; padding: 10px; font-family: monospace; }
    </style>
</head>
<body>
    <h1>🔓 漏洞商品搜索系统</h1>
    
    <div class="vulnerable">
        <h2>🚨 危险搜索（SQL注入点）</h2>
        <form method="GET" action="/">
            <input type="text" name="search" placeholder="搜索商品..." 
                   value="{{ search_term or '' }}" size="50">
            <button type="submit">搜索</button>
        </form>
        
        {% if sql_query %}
        <div class="sql">
            <strong>执行的SQL:</strong><br>
            {{ sql_query }}
        </div>
        {% endif %}
        
        {% if error %}
        <div class="error">
            <strong>错误:</strong> {{ error }}
        </div>
        {% endif %}
    </div>
    
    <div class="safe">
        <h2>🛡️ 安全搜索</h2>
        <form method="GET" action="/safe">
            <input type="text" name="search" placeholder="安全搜索商品..." 
                   value="{{ safe_search or '' }}" size="50">
            <button type="submit">安全搜索</button>
        </form>
    </div>
    
    {% if products %}
    <h3>搜索结果 ({{ products|length }} 个商品)</h3>
    {% for product in products %}
    <div class="product">
        <h4>{{ product.name }}</h4>
        <p>价格: ¥{{ "%.2f"|format(product.price) }}</p>
        <p>{{ product.description }}</p>
    </div>
    {% endfor %}
    {% elif request.path == '/' and search_term %}
    <p>没有找到商品。</p>
    {% endif %}
    
    <hr>
    <h3>🧪 攻击测试用例</h3>
    <p>在危险搜索框中尝试输入：</p>
    <ul>
        <li><code>' OR '1'='1</code> - 显示所有商品</li>
        <li><code>'; DELETE FROM products; --</code> - 删除所有商品（危险！）</li>
        <li><code>' UNION SELECT id, username, password_hash, null FROM users --</code> - 获取用户信息</li>
        <li><code>' AND 1=0 UNION SELECT 1,2,3,4 --</code> - 测试字段数</li>
    </ul>
</body>
</html>
'''

@app.route('/')
def vulnerable_search():
    """危险的搜索功能 - 存在SQL注入"""
    search_term = request.args.get('search', '')
    products = []
    sql_query = ""
    error = ""
    
    if search_term:
        conn = get_db_connection()
        try:
            # 🚨 危险！直接拼接用户输入
            sql = f"SELECT * FROM products WHERE name LIKE '%{search_term}%' OR description LIKE '%{search_term}%'"
            sql_query = sql
            
            print(f"[危险] 执行SQL: {sql}")  # 控制台输出
            
            cursor = conn.execute(sql)
            products = cursor.fetchall()
        except sqlite3.Error as e:
            error = str(e)
            print(f"[错误] SQL错误: {e}")
        finally:
            conn.close()
    
    return render_template_string(HTML_TEMPLATE, 
                                 search_term=search_term,
                                 products=products,
                                 sql_query=sql_query,
                                 error=error)

@app.route('/safe')
def safe_search():
    """安全的搜索功能"""
    search_term = request.args.get('search', '')
    products = []
    
    if search_term:
        conn = get_db_connection()
        try:
            # 🟢 安全：使用参数化查询
            sql = "SELECT * FROM products WHERE name LIKE ? OR description LIKE ?"
            search_pattern = f"%{search_term}%"
            
            print(f"[安全] 执行SQL: {sql}")  # 控制台输出
            print(f"[安全] 参数: {search_pattern}")
            
            cursor = conn.execute(sql, (search_pattern, search_pattern))
            products = cursor.fetchall()
        except sqlite3.Error as e:
            print(f"[安全] 错误: {e}")
        finally:
            conn.close()
    
    return render_template_string(HTML_TEMPLATE, 
                                 safe_search=search_term,
                                 products=products)

if __name__ == '__main__':
    print("🌐 启动漏洞演示Web应用...")
    print("访问: http://localhost:5000")
    print("危险搜索: http://localhost:5000/?search=手机")
    print("安全搜索: http://localhost:5000/safe?search=手机")
    print("-" * 50)
    app.run(debug=True, port=5000)
```

**运行与测试**：
```bash
python 02_flask_vulnerable_app.py
```

**学习任务**：
1. 访问 `http://localhost:5000`
2. 尝试所有攻击用例
3. 观察控制台输出的SQL语句
4. 比较危险搜索和安全搜索的结果差异
5. **重要**：尝试`' UNION SELECT id, username, password_hash, null FROM users --`获取用户数据

---

## 项目3：用户登录系统（含漏洞和安全版）
**目标**：综合应用，实现完整登录系统

### `03_login_system.py`
```python
from flask import Flask, request, render_template_string, redirect, url_for, session
import sqlite3
import bcrypt
import html

app = Flask(__name__)
app.secret_key = 'dev-secret-key-123'  # 生产环境要用强密钥
DATABASE = 'vulnerable_app.db'

def get_db_connection():
    conn = sqlite3.connect(DATABASE)
    conn.row_factory = sqlite3.Row
    return conn

# 登录页面HTML
LOGIN_HTML = '''
<!DOCTYPE html>
<html>
<head>
    <title>登录系统 - 漏洞演示</title>
    <style>
        body { font-family: Arial; max-width: 600px; margin: 50px auto; }
        .tab { overflow: hidden; border: 1px solid #ccc; background: #f1f1f1; }
        .tab button { background: inherit; float: left; border: none; outline: none; 
                     padding: 14px 16px; cursor: pointer; transition: 0.3s; }
        .tab button:hover { background: #ddd; }
        .tab button.active { background: #ccc; }
        .tabcontent { padding: 20px; border: 1px solid #ccc; border-top: none; }
        .vulnerable { background: #ffe6e6; }
        .safe { background: #e6ffe6; }
        input { padding: 10px; margin: 8px 0; width: 95%; }
        button { background: #4CAF50; color: white; padding: 12px; border: none; cursor: pointer; }
        .error { color: red; padding: 10px; }
        .success { color: green; padding: 10px; }
        .user-info { background: #f0f8ff; padding: 15px; margin: 20px 0; }
    </style>
</head>
<body>
    <h1>🔐 登录系统安全对比</h1>
    
    {% if session.username %}
    <div class="user-info">
        <h3>👤 已登录用户: {{ session.username }}</h3>
        <p>邮箱: {{ session.email }}</p>
        <a href="/logout"><button>退出登录</button></a>
    </div>
    {% endif %}
    
    <div class="tab">
        <button class="tablinks active" onclick="openTab(event, 'Vulnerable')">🚨 漏洞登录</button>
        <button class="tablinks" onclick="openTab(event, 'Safe')">🛡️ 安全登录</button>
        <button class="tablinks" onclick="openTab(event, 'Demo')">🧪 攻击演示</button>
    </div>
    
    <div id="Vulnerable" class="tabcontent vulnerable">
        <h2>危险登录系统</h2>
        <form method="POST" action="/vulnerable_login">
            <input type="text" name="username" placeholder="用户名" required><br>
            <input type="password" name="password" placeholder="密码" required><br>
            <button type="submit">登录</button>
        </form>
        
        {% if vulnerable_error %}
        <div class="error">{{ vulnerable_error }}</div>
        {% endif %}
        
        {% if vulnerable_sql %}
        <div style="background:#f5f5f5; padding:10px; margin-top:15px;">
            <strong>执行的SQL:</strong><br>
            <code>{{ vulnerable_sql }}</code>
        </div>
        {% endif %}
    </div>
    
    <div id="Safe" class="tabcontent safe" style="display:none;">
        <h2>安全登录系统</h2>
        <form method="POST" action="/safe_login">
            <input type="text" name="username" placeholder="用户名" required><br>
            <input type="password" name="password" placeholder="密码" required><br>
            <button type="submit">安全登录</button>
        </form>
        
        {% if safe_error %}
        <div class="error">{{ safe_error }}</div>
        {% endif %}
    </div>
    
    <div id="Demo" class="tabcontent" style="display:none;">
        <h2>攻击演示</h2>
        <p>在漏洞登录系统中尝试：</p>
        
        <div style="background:#fff3cd; padding:15px; margin:10px 0;">
            <h4>🧨 攻击1: 绕过密码验证</h4>
            <p>用户名: <code>alice' --</code></p>
            <p>密码: <em>任意值</em></p>
            <button onclick="fillForm('alice\\' --', 'anything')">自动填充</button>
        </div>
        
        <div style="background:#fff3cd; padding:15px; margin:10px 0;">
            <h4>🧨 攻击2: 获取所有用户权限</h4>
            <p>用户名: <code>' OR '1'='1' --</code></p>
            <p>密码: <em>任意值</em></p>
            <button onclick="fillForm('\\' OR \\'1\\'=\\'1\\' --', 'anything')">自动填充</button>
        </div>
        
        <div style="background:#fff3cd; padding:15px; margin:10px 0;">
            <h4>🧨 攻击3: 联合查询获取数据</h4>
            <p>用户名: <code>' UNION SELECT 1, username, password_hash, email FROM users --</code></p>
            <p>密码: <em>任意值</em></p>
            <button onclick="fillForm('\\' UNION SELECT 1, username, password_hash, email FROM users --', 'anything')">自动填充</button>
        </div>
    </div>
    
    <script>
    function openTab(evt, tabName) {
        var i, tabcontent, tablinks;
        tabcontent = document.getElementsByClassName("tabcontent");
        for (i = 0; i < tabcontent.length; i++) {
            tabcontent[i].style.display = "none";
        }
        tablinks = document.getElementsByClassName("tablinks");
        for (i = 0; i < tablinks.length; i++) {
            tablinks[i].className = tablinks[i].className.replace(" active", "");
        }
        document.getElementById(tabName).style.display = "block";
        evt.currentTarget.className += " active";
    }
    
    function fillForm(username, password) {
        document.querySelector('[name="username"]').value = username;
        document.querySelector('[name="password"]').value = password;
        document.querySelector('.tablinks').click(); // 切换到漏洞标签
    }
    </script>
</body>
</html>
'''

@app.route('/')
def index():
    """主页面"""
    return render_template_string(LOGIN_HTML,
                                 vulnerable_error=request.args.get('v_error'),
                                 vulnerable_sql=request.args.get('v_sql'),
                                 safe_error=request.args.get('s_error'),
                                 session=session)

@app.route('/vulnerable_login', methods=['POST'])
def vulnerable_login():
    """危险的登录处理"""
    username = request.form['username']
    password = request.form['password']
    
    conn = get_db_connection()
    
    # 🚨 危险：直接拼接SQL（且错误地比较密码哈希）
    sql = f"SELECT * FROM users WHERE username = '{username}' AND password_hash = '{password}'"
    
    try:
        cursor = conn.execute(sql)
        user = cursor.fetchone()
        
        if user:
            # 登录成功（实际上应该是验证密码哈希）
            session['username'] = user['username']
            session['email'] = user['email']
            return redirect(url_for('index'))
        else:
            return redirect(url_for('index', 
                                  v_error='用户名或密码错误',
                                  v_sql=html.escape(sql)))
    except Exception as e:
        return redirect(url_for('index', 
                              v_error=f'SQL错误: {e}',
                              v_sql=html.escape(sql)))
    finally:
        conn.close()

@app.route('/safe_login', methods=['POST'])
def safe_login():
    """安全的登录处理"""
    username = request.form['username']
    password = request.form['password']
    
    conn = get_db_connection()
    
    # 🟢 安全：参数化查询
    sql = "SELECT password_hash, username, email FROM users WHERE username = ?"
    
    try:
        cursor = conn.execute(sql, (username,))
        user = cursor.fetchone()
        
        if user and bcrypt.checkpw(password.encode(), user['password_hash'].encode()):
            # 密码验证正确
            session['username'] = user['username']
            session['email'] = user['email']
            return redirect(url_for('index'))
        else:
            return redirect(url_for('index', s_error='用户名或密码错误'))
    except Exception as e:
        return redirect(url_for('index', s_error=f'登录错误: {e}'))
    finally:
        conn.close()

@app.route('/logout')
def logout():
    """退出登录"""
    session.clear()
    return redirect(url_for('index'))

if __name__ == '__main__':
    print("🔐 启动登录系统演示...")
    print("访问: http://localhost:5001")
    print("-" * 50)
    app.run(debug=True, port=5001)
```

**运行与测试**：
```bash
python 03_login_system.py
```

**核心学习点**：
1. **会话管理**：Flask的`session`使用
2. **密码安全**：`bcrypt.checkpw()`验证哈希密码
3. **SQL注入对比**：直接拼接 vs 参数化查询
4. **输出安全**：使用`html.escape()`防止XSS

---

## 项目4：使用SQLAlchemy的安全实践
**目标**：学习使用ORM框架，理解其安全性

### `04_sqlalchemy_safe.py`
```python
from flask import Flask, request, render_template_string
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy import text
import html

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///vulnerable_app.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

# 定义数据模型
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), unique=True, nullable=False)
    email = db.Column(db.String(100), unique=True, nullable=False)
    password_hash = db.Column(db.String(255), nullable=False)

class Product(db.Model):
    __tablename__ = 'products'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    price = db.Column(db.Float, nullable=False)
    description = db.Column(db.Text)

# 创建表（如果不存在）
with app.app_context():
    db.create_all()

HTML_TEMPLATE = '''
<!DOCTYPE html>
<html>
<head>
    <title>SQLAlchemy 安全实践</title>
</head>
<body>
    <h1>SQLAlchemy 查询方式对比</h1>
    
    <h2>🔍 搜索产品</h2>
    <form method="GET">
        <input type="text" name="query" placeholder="产品名称">
        <select name="method">
            <option value="safe_orm">🟢 安全ORM查询</option>
            <option value="safe_text">🟡 安全Text查询</option>
            <option value="vulnerable">🔴 危险字符串拼接</option>
        </select>
        <button type="submit">搜索</button>
    </form>
    
    {% if query %}
    <h3>搜索 "{{ query }}" 的结果 ({{ method_name }})</h3>
    {% if sql %}
    <div style="background:#f5f5f5; padding:10px; margin:10px 0;">
        <strong>SQL语句:</strong><br>
        <code>{{ sql }}</code>
    </div>
    {% endif %}
    
    {% if products %}
    <ul>
        {% for product in products %}
        <li>{{ product.name }} - ¥{{ "%.2f"|format(product.price) }}</li>
        {% endfor %}
    </ul>
    {% else %}
    <p>没有找到产品。</p>
    {% endif %}
    {% endif %}
    
    <hr>
    <h3>🎓 学习要点</h3>
    <ol>
        <li><strong>安全ORM查询</strong>: 使用filter方法，SQLAlchemy自动参数化</li>
        <li><strong>安全Text查询</strong>: 使用text()和命名参数</li>
        <li><strong>危险拼接</strong>: 直接拼接字符串，存在SQL注入</li>
    </ol>
</body>
</html>
'''

@app.route('/')
def index():
    query = request.args.get('query', '')
    method = request.args.get('method', 'safe_orm')
    
    products = []
    sql_statement = ""
    method_name = ""
    
    if query:
        if method == 'safe_orm':
            method_name = "安全ORM查询"
            # 🟢 安全：使用ORM的filter方法
            products = Product.query.filter(Product.name.contains(query)).all()
            # SQLAlchemy会自动生成参数化查询
            
        elif method == 'safe_text':
            method_name = "安全Text查询"
            # 🟡 较安全：使用text()和命名参数
            sql = text("SELECT * FROM products WHERE name LIKE :pattern")
            result = db.session.execute(sql, {'pattern': f'%{query}%'})
            products = [Product(**dict(row)) for row in result.mappings()]
            sql_statement = str(sql)
            
        elif method == 'vulnerable':
            method_name = "危险字符串拼接"
            # 🚨 危险：直接拼接（演示用，不要在生产环境使用！）
            vulnerable_sql = f"SELECT * FROM products WHERE name LIKE '%{query}%'"
            try:
                result = db.session.execute(text(vulnerable_sql))
                products = [Product(**dict(row)) for row in result.mappings()]
                sql_statement = vulnerable_sql
            except Exception as e:
                sql_statement = f"{vulnerable_sql}<br><strong>错误:</strong> {e}"
    
    return render_template_string(HTML_TEMPLATE,
                                 query=html.escape(query),
                                 method_name=method_name,
                                 sql=sql_statement,
                                 products=products)

if __name__ == '__main__':
    print("🧪 启动SQLAlchemy安全演示...")
    print("访问: http://localhost:5002")
    print("-" * 50)
    
    # 测试不同查询方法的安全性
    print("\n测试用例:")
    print("1. 正常搜索: 电脑")
    print("2. SQL注入尝试: ' OR '1'='1")
    print("3. 联合查询: ' UNION SELECT * FROM users --")
    
    app.run(debug=True, port=5002)
```

**学习要点**：
1. **ORM的优势**：自动参数化，防止SQL注入
2. **text()查询**：需要显式使用参数绑定
3. **永远不要拼接**：直接拼接字符串是危险的

---

## 📊 学习路径总结

| 项目 | 核心技能 | 安全重点 | 建议学习时间 |
|-----|---------|---------|------------|
| 项目1 | Python基础，SQLite操作 | SQL注入原理理解 | 2-3小时 |
| 项目2 | Flask Web基础 | Web环境下的SQL注入 | 3-4小时 |
| 项目3 | 会话管理，密码安全 | 完整登录系统的安全 | 4-5小时 |
| 项目4 | SQLAlchemy ORM | ORM框架的安全性 | 3-4小时 |

## 🔧 进阶挑战

完成基础项目后，尝试这些挑战：

### 挑战1：盲注攻击演示
创建一个页面，通过布尔盲注或时间盲注获取数据。

### 挑战2：自动化SQL注入工具
写一个Python脚本，自动检测SQL注入漏洞。

### 挑战3：完整博客系统
实现一个有用户、文章、评论的博客系统，包含所有安全措施。

## 🚀 立即开始

建议的学习顺序：
1. `python init_database.py` - 初始化数据库
2. `python 01_basic_injection.py` - 理解基础原理
3. `python 02_flask_vulnerable_app.py` - 在Web环境中实践
4. 分别在不同终端运行项目2、3、4，对比学习

**记住**：在学习过程中，要故意制造漏洞，然后修复它们。这样你才能真正理解安全问题。

遇到具体问题时，随时可以问！
