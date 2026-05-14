# Pruebas Funcionales Automatizadas con pytest

## Red Velvet - Sistema de Gestión de Restaurante

---

## 1. Información General

| Ítem | Descripción |
|------|-------------|
| **Aplicación** | Red Velvet - Sistema de Gestión de Restaurante |
| **Frontend** | https://frontend-chi-five-62.vercel.app |
| **API** | https://loyal-smile-production-2893.up.railway.app/api/v1 |
| **Herramientas** | pytest + requests (API) + selenium (UI) |
| **Lenguaje** | Python 3.9+ |

---

## 2. Configuración del Entorno

### 2.1 Requisitos
- Python 3.9+
- Google Chrome o Chromium
- pip (gestor de paquetes de Python)

### 2.2 Instalación
```bash
# Crear estructura de directorios
mkdir -p tests/pytest tests/evidencias

# Instalar dependencias
pip3 install pytest requests selenium pytest-html
```

### 2.3 Estructura del Proyecto
```
tests/pytest/
├── pytest.ini                 # Configuración de pytest
├── config.py                  # Constantes compartidas (URLs, credenciales)
├── conftest.py                # Fixtures (tokens, productos de prueba)
├── test_api.py                # 18 pruebas de API
├── test_ui.py                 # 8 pruebas de interfaz de usuario
├── report.html                # Reporte HTML generado
tests/evidencias/              # Capturas de pantalla
    ├── pytest-login-admin.png
    ├── pytest-login-cliente.png
    ├── pytest-login-mesero.png
    ├── pytest-login-fallido.png
    ├── pytest-menu-publico.png
    ├── pytest-menu-filtro.png
    ├── pytest-carrito.png
    └── pytest-registro.png
```

---

## 3. Casos de Prueba

### 3.1 Pruebas de API (test_api.py)

| # | Clase | Prueba | Descripción |
|---|-------|--------|-------------|
| 1 | TestAuth | `test_login_admin_exitoso` | Login admin retorna token y rol correcto |
| 2 | TestAuth | `test_login_cliente_exitoso` | Login cliente retorna rol=cliente |
| 3 | TestAuth | `test_login_mesero_exitoso` | Login mesero retorna rol=mesero |
| 4 | TestAuth | `test_login_credenciales_invalidas` | Login fallido retorna 401 |
| 5 | TestAuth | `test_get_profile` | Perfil con token devuelve email correcto |
| 6 | TestAuth | `test_get_profile_sin_token` | Perfil sin token retorna 401 |
| 7 | TestCatalogos | `test_listar_categorias` | GET /categorias retorna array con "Entradas" |
| 8 | TestCatalogos | `test_listar_productos` | GET /products retorna array |
| 9 | TestCatalogos | `test_listar_mesas` | GET /tables retorna 8 mesas |
| 10 | TestCatalogos | `test_listar_toppings` | GET /toppings contiene "Aguacate" |
| 11 | TestProductosCRUD | `test_crear_producto` | POST /products crea producto |
| 12 | TestProductosCRUD | `test_obtener_producto_por_id` | GET /products/:id retorna producto |
| 13 | TestProductosCRUD | `test_actualizar_producto` | PUT /products/:id actualiza nombre |
| 14 | TestProductosCRUD | `test_desactivar_producto` | DELETE /products/:id desactiva |
| 15 | TestProductosCRUD | `test_crear_producto_sin_nombre` | POST sin nombre (caso borde) |
| 16 | TestProductosCRUD | `test_crear_producto_sin_autenticacion` | POST sin token retorna 401 |
| 17 | TestOrdenes | `test_crear_orden_local` | POST /orders crea pedido local |
| 18 | TestHealth | `test_health_check` | GET /health retorna status=ok |

### 3.2 Pruebas de UI (test_ui.py)

| # | Clase | Prueba | Descripción |
|---|-------|--------|-------------|
| 19 | TestLogin | `test_login_admin_redirige_a_admin` | Login admin redirige a /admin |
| 20 | TestLogin | `test_login_cliente_redirige_a_cliente` | Login cliente redirige a /cliente |
| 21 | TestLogin | `test_login_mesero_redirige_a_mesero` | Login mesero redirige a /mesero |
| 22 | TestLogin | `test_login_fallido_muestra_error` | Login fallido muestra mensaje |
| 23 | TestMenuPublico | `test_menu_muestra_productos` | Menú carga al menos 1 producto |
| 24 | TestMenuPublico | `test_filtro_por_categoria` | Filtro por categoría funciona |
| 25 | TestCarrito | `test_agregar_producto_al_carrito` | Agregar producto al carrito |
| 26 | TestRegistro | `test_formulario_registro_visible` | Formulario de registro visible |

---

## 4. Scripts de Prueba

### 4.1 Configuración Común (`config.py`)
```python
API_BASE = "https://loyal-smile-production-2893.up.railway.app/api/v1"
FRONTEND = "https://frontend-chi-five-62.vercel.app"

CREDENTIALS = {
    "admin": {"email": "admin@redvelvet.com", "password": "admin123"},
    "cliente": {"email": "cliente@redvelvet.com", "password": "cliente123"},
    "mesero": {"email": "mesero1@redvelvet.com", "password": "mesero123"},
    "master": {"email": "master@redvelvet.com", "password": "master123"},
}
```

### 4.2 Ejemplo: Prueba de Login Admin
```python
def test_login_admin_exitoso(self):
    r = requests.post(f"{API_BASE}/auth/login", json=CREDENTIALS["admin"])
    assert r.status_code == 200
    data = r.json()
    assert "accessToken" in data
    assert data["user"]["rol"] == "administrador"
```

### 4.3 Ejemplo: Prueba de UI con Selenium
```python
def test_login_admin_redirige_a_admin(self, driver):
    driver.get(f"{FRONTEND}/login")
    WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.CSS_SELECTOR, "input[type='email']"))
    )
    driver.find_element(By.CSS_SELECTOR, "input[type='email']").send_keys(
        CREDENTIALS["admin"]["email"]
    )
    driver.find_element(By.CSS_SELECTOR, "input[type='password']").send_keys(
        CREDENTIALS["admin"]["password"]
    )
    driver.find_element(By.CSS_SELECTOR, "button[type='submit']").click()
    import time; time.sleep(4)
    assert "/admin" in driver.current_url
```

---

## 5. Ejecución

```bash
cd tests/pytest

# Ejecutar todas las pruebas
python3 -m pytest -v

# Ejecutar solo pruebas de API
python3 -m pytest test_api.py -v

# Ejecutar solo pruebas de UI
python3 -m pytest test_ui.py -v

# Generar reporte HTML
python3 -m pytest --html=report.html --self-contained-html
```

### Resultado Obtenido
```
collected 26 items

test_api.py::TestAuth::test_login_admin_exitoso ✓
test_api.py::TestAuth::test_login_cliente_exitoso ✓
test_api.py::TestAuth::test_login_mesero_exitoso ✓
test_api.py::TestAuth::test_login_credenciales_invalidas ✓
test_api.py::TestAuth::test_get_profile ✓
test_api.py::TestAuth::test_get_profile_sin_token ✓
test_api.py::TestCatalogos::test_listar_categorias ✓
test_api.py::TestCatalogos::test_listar_productos ✓
test_api.py::TestCatalogos::test_listar_mesas ✓
test_api.py::TestCatalogos::test_listar_toppings ✓
test_api.py::TestProductosCRUD::test_crear_producto ✓
test_api.py::TestProductosCRUD::test_obtener_producto_por_id ✓
test_api.py::TestProductosCRUD::test_actualizar_producto ✓
test_api.py::TestProductosCRUD::test_desactivar_producto ✓
test_api.py::TestProductosCRUD::test_crear_producto_sin_nombre ✓
test_api.py::TestProductosCRUD::test_crear_producto_sin_autenticacion ✓
test_api.py::TestOrdenes::test_crear_orden_local ✓
test_api.py::TestHealth::test_health_check ✓
test_ui.py::TestLogin::test_login_admin_redirige_a_admin ✓
test_ui.py::TestLogin::test_login_cliente_redirige_a_cliente ✓
test_ui.py::TestLogin::test_login_mesero_redirige_a_mesero ✓
test_ui.py::TestLogin::test_login_fallido_muestra_error ✓
test_ui.py::TestMenuPublico::test_menu_muestra_productos ✓
test_ui.py::TestMenuPublico::test_filtro_por_categoria ✓
test_ui.py::TestCarrito::test_agregar_producto_al_carrito ✓
test_ui.py::TestRegistro::test_formulario_registro_visible ✓

26 passed in 66.30s
```

---

## 6. Evidencias

### Capturas de Pantalla (UI Tests)
| Captura | Prueba | Descripción |
|---------|--------|-------------|
| `pytest-login-admin.png` | Login Admin | Dashboard de administrador |
| `pytest-login-cliente.png` | Login Cliente | Panel de cliente |
| `pytest-login-mesero.png` | Login Mesero | Panel de mesero |
| `pytest-login-fallido.png` | Login Inválido | Mensaje de error mostrado |
| `pytest-menu-publico.png` | Menú Público | Productos visibles |
| `pytest-menu-filtro.png` | Filtro Categoría | Productos filtrados |
| `pytest-carrito.png` | Carrito | Ítem en carrito |
| `pytest-registro.png` | Registro | Formulario visible |

### Reporte HTML
El archivo `tests/pytest/report.html` contiene un reporte interactivo con:
- Resumen de todas las pruebas (26/26 pasaron)
- Tiempo de ejecución de cada prueba
- Trazas de error detalladas
- Metadata del entorno

---

## 7. Análisis de Resultados

### 7.1 Métricas

| Métrica | Valor |
|---------|-------|
| Total de pruebas | 26 |
| Pasaron | 26 (100%) |
| Fallaron | 0 |
| Tiempo total | 66.30s |
| Pruebas API | 18 (69%) |
| Pruebas UI | 8 (31%) |

### 7.2 Cobertura por Funcionalidad

| Funcionalidad | Pruebas | Estado |
|---------------|---------|--------|
| Autenticación (login 3 roles + fallido + perfil) | 6 | ✅ |
| Control de acceso (token requerido) | 2 | ✅ |
| Catálogos (categorías, productos, mesas, toppings) | 4 | ✅ |
| CRUD Productos (crear, leer, actualizar, desactivar) | 4 | ✅ |
| Órdenes (crear pedido local) | 1 | ✅ |
| Health Check | 1 | ✅ |
| Navegación UI (login + redirección) | 4 | ✅ |
| Menú UI (carga + filtro) | 2 | ✅ |
| Carrito UI | 1 | ✅ |
| Registro UI | 1 | ✅ |

### 7.3 Casos de Prueba Críticos
1. **Login**: Verifica que cada rol (admin, cliente, mesero) puede iniciar sesión y es redirigido al panel correcto
2. **Control de acceso**: Confirma que endpoints protegidos rechazan peticiones sin token
3. **CRUD**: Ciclo completo de crear, leer, actualizar y desactivar un producto
4. **Flujo de negocio**: Creación de pedido local con productos
5. **Disponibilidad del sistema**: Health check del servidor

---

## 8. Reflexión Grupal

### 8.1 ¿Qué aprendimos?

- **pytest** es un framework de testing maduro y flexible que permite organizar pruebas en clases y usar fixtures para compartir recursos (como tokens de autenticación) entre pruebas.
- **requests** es una librería simple pero potente para probar APIs REST. Con pocas líneas podemos verificar status codes, campos JSON y autenticación.
- **Selenium WebDriver** desde Python permite automatizar navegadores reales para probar la interfaz de usuario. Las pruebas UI son más lentas pero capturan problemas que las pruebas de API no pueden detectar (como redirecciones incorrectas, elementos que no renderizan, etc.).
- La combinación de pytest + requests + selenium da cobertura completa: validamos la lógica del backend y la experiencia del usuario final.

### 8.2 Dificultades Encontradas

1. **Aplicaciones SPA (React)**: Las redirecciones con `window.location.href` no siempre son detectadas por Selenium. Se resolvió usando `time.sleep()` con tiempos generosos y navegación directa como fallback.
2. **Selectores dinámicos**: Los componentes de React pueden tener clases CSS generadas dinámicamente. Usamos selectores XPath basados en texto (`contains(text(), '...')`) que son más estables.
3. **Headless Chrome**: Algunos comportamientos (modales, popups) difieren entre modo headless y modo gráfico. Las pruebas se ajustaron para manejar ambas situaciones.
4. **Manejo de estado compartido**: Las pruebas de API comparten tokens y IDs de productos. pytest fixtures con scope "session" y "function" ayudan a gestionar el ciclo de vida.

### 8.3 Mejoras Propuestas

1. **Data-Driven Testing**: Parametrizar pruebas con `@pytest.mark.parametrize` para ejecutar el mismo test con múltiples conjuntos de datos.
2. **Page Object Model (POM)**: Organizar los selectores de UI en clases separadas para facilitar el mantenimiento.
3. **CI/CD Integration**: Ejecutar `pytest --html=report.xml --junitxml=results.xml` en GitHub Actions para generar reportes automáticos en cada push.
4. **Pruebas de estrés**: Usar `pytest-benchmark` para medir tiempos de respuesta de la API bajo carga.
5. **Capturas automáticas en fallos**: Configurar `pytest` para tomar screenshot automáticamente cuando una prueba UI falla.

### 8.4 Conclusión

Las 26 pruebas funcionales automatizadas (18 de API + 8 de UI) demostraron que el sistema Red Velvet se encuentra en un estado estable y funcional. La combinación de pytest + requests para backend y pytest + selenium para frontend proporciona una cobertura completa que permite detectar regresiones rápidamente. El reporte HTML generado facilita la revisión de resultados por parte de todo el equipo.

---

## 9. Cómo Reproducir

```bash
# 1. Clonar el repositorio
git clone https://github.com/andresjc159-dotcom/sigr.git
cd sigr

# 2. Instalar dependencias de Python
pip3 install pytest requests selenium pytest-html

# 3. Ejecutar todas las pruebas
python3 -m pytest tests/pytest/ -v

# 4. Generar reporte HTML
python3 -m pytest tests/pytest/ --html=tests/pytest/report.html --self-contained-html

# 5. Ver evidencias
ls tests/evidencias/
```
