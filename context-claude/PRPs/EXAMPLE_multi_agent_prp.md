nombre: "Plantilla Base PRP v2 - Rica en Contexto con Bucles de Validación"
descripción: |

## Propósito
Plantilla optimizada para que los agentes de IA implementen funcionalidades con suficiente contexto y capacidades de autovalidación para lograr un código funcional a través de refinamiento iterativo.

## Principios Fundamentales
1.  **El Contexto es Clave**: Incluir TODA la documentación, ejemplos y advertencias necesarias.
2.  **Bucles de Validación**: Proporcionar pruebas/análisis estáticos (lints) ejecutables que la IA pueda correr y corregir.
3.  **Denso en Información**: Usar palabras clave y patrones de la base de código.
4.  **Éxito Progresivo**: Empezar de forma simple, validar y luego mejorar.
5.  **Reglas Globales**: Asegúrese de seguir todas las reglas en CLAUDE.md.

---

## Objetivo
[Qué se necesita construir - sea específico sobre el estado final y los objetivos deseados]

## Por qué
- [Valor para el negocio e impacto en el usuario]
- [Integración con funcionalidades existentes]
- [Problemas que esto resuelve y para quién]

## Qué
[Comportamiento visible para el usuario y requisitos técnicos]

### Criterios de Éxito
- [ ] [Resultados específicos y medibles]

## Todo el Contexto Necesario

### Documentación y Referencias (liste todo el contexto necesario para implementar la funcionalidad)
```yaml
# LECTURA OBLIGATORIA - Incluya esto en su ventana de contexto
- url: [https://api.www.documentcloud.org/api](https://api.www.documentcloud.org/api)
  por_que: [Secciones/métodos específicos que necesitará]

- file: [ruta/a/ejemplo.py]
  por_que: [Patrón a seguir, problemas a evitar]

- doc: [https://www.sigmaaldrich.com/docs](https://www.sigmaaldrich.com/docs)
  seccion: [Sección específica sobre errores comunes]
  critico: [Idea clave que previene errores comunes]

- docfile: [PRPs/ai_docs/archivo.md]
  por_que: [documentos que el usuario ha pegado en el proyecto]
```

### Árbol actual de la base de código (ejecute `tree` en la raíz del proyecto) para obtener una visión general
```bash

```

### Árbol deseado de la base de código con los archivos a añadir y la responsabilidad de cada uno
```bash

```

### Problemas Conocidos de nuestra base de código y Particularidades de las Librerías
```python
# CRÍTICO: La librería [nombre] requiere [configuración específica]
# Ejemplo: FastAPI requiere funciones asíncronas para los endpoints
# Ejemplo: Este ORM no soporta inserciones por lotes de más de 1000 registros
# Ejemplo: Usamos pydantic v2 y
```

## Plan de Implementación

### Modelos y estructura de datos

Crear los modelos de datos principales; aseguramos la seguridad de tipos (type safety) y la consistencia.
```python
Ejemplos:
 - modelos ORM
 - modelos pydantic
 - esquemas pydantic
 - validadores pydantic
```

### Lista de tareas a completar para cumplir el PRP en el orden en que deben realizarse

```yaml
Tarea 1:
MODIFICAR src/modulo_existente.py:
  - BUSCAR patrón: "class ImplementacionAntigua"
  - INYECTAR después de la línea que contiene "def __init__"
  - CONSERVAR las firmas de los métodos existentes

CREAR src/nueva_funcionalidad.py:
  - REPLICAR patrón de: src/funcionalidad_similar.py
  - MODIFICAR el nombre de la clase y la lógica principal
  - MANTENER idéntico el patrón de manejo de errores

...(...)

Tarea N:
...
```

### Pseudocódigo por tarea según sea necesario añadido a cada tarea
```python

# Tarea 1
# Pseudocódigo con detalles CRÍTICOS, no escribir el código completo
async def nueva_funcionalidad(param: str) -> Result:
    # PATRÓN: Siempre valide la entrada primero (ver src/validators.py)
    validated = validate_input(param)  # lanza ValidationError

    # OJO: Esta librería requiere un pool de conexiones (connection pooling)
    async with get_connection() as conn:  # ver src/db/pool.py
        # PATRÓN: Use el decorador de reintentos existente
        @retry(attempts=3, backoff=exponential)
        async def _inner():
            # CRÍTICO: La API devuelve 429 si hay >10 req/seg
            await rate_limiter.acquire()
            return await external_api.call(validated)

        result = await _inner()

    # PATRÓN: Formato de respuesta estandarizado
    return format_response(result)  # ver src/utils/responses.py
```

### Puntos de Integración
```yaml
BASE_DE_DATOS:
  - migracion: "Añadir columna 'feature_enabled' a la tabla de usuarios"
  - indice: "CREATE INDEX idx_feature_lookup ON users(feature_id)"

CONFIG:
  - añadir a: config/settings.py
  - patron: "FEATURE_TIMEOUT = int(os.getenv('FEATURE_TIMEOUT', '30'))"

RUTAS:
  - añadir a: src/api/routes.py
  - patron: "router.include_router(feature_router, prefix='/feature')"
```

## Bucle de Validación

### Nivel 1: Sintaxis y Estilo
```bash
# Ejecute esto PRIMERO - corrija cualquier error antes de proceder
ruff check src/nueva_funcionalidad.py --fix  # Corregir automáticamente lo que sea posible
mypy src/nueva_funcionalidad.py             # Verificación de tipos

# Esperado: Sin errores. Si hay errores, LEA el error y corríjalo.
```

### Nivel 2: Pruebas Unitarias para cada nueva funcionalidad/archivo/función usando los patrones existentes
```python
# CREAR test_nueva_funcionalidad.py con estos casos de prueba:
def test_camino_feliz():
    """La funcionalidad básica funciona"""
    result = nueva_funcionalidad("entrada_valida")
    assert result.status == "success"

def test_error_de_validacion():
    """Una entrada inválida lanza ValidationError"""
    with pytest.raises(ValidationError):
        nueva_funcionalidad("")

def test_timeout_api_externa():
    """Maneja los tiempos de espera (timeouts) correctamente"""
    with mock.patch('external_api.call', side_effect=TimeoutError):
        result = nueva_funcionalidad("valido")
        assert result.status == "error"
        assert "timeout" in result.message
```bash
# Ejecutar e iterar hasta que pasen:
uv run pytest test_nueva_funcionalidad.py -v
# Si falla: Lea el error, entienda la causa raíz, corrija el código y vuelva a ejecutar (nunca haga un mock para que la prueba pase).
```

### Nivel 3: Pruebas de Integración
```bash
# Iniciar el servicio
uv run python -m src.main --dev

# Probar el endpoint
curl -X POST http://localhost:8000/feature \
  -H "Content-Type: application/json" \
  -d '{"param": "valor_de_prueba"}'

# Esperado: {"status": "success", "data": {...}}
# Si hay error: Revise los logs en logs/app.log para ver el stack trace.
```

## Lista de Verificación Final
- [ ] Todas las pruebas pasan: `uv run pytest tests/ -v`
- [ ] Sin errores de linting: `uv run ruff check src/`
- [ ] Sin errores de tipo: `uv run mypy src/`
- [ ] Prueba manual exitosa: [curl/comando específico]
- [ ] Casos de error manejados correctamente
- [ ] Los logs son informativos pero no verbosos
- [ ] Documentación actualizada si es necesario

---

## Antipatrones a Evitar
- ❌ No cree nuevos patrones cuando los existentes funcionan.
- ❌ No omita la validación porque "debería funcionar".
- ❌ No ignore las pruebas que fallan; corríjalas.
- ❌ No use funciones síncronas en un contexto asíncrono.
- ❌ No inserte valores fijos en el código que deberían estar en la configuración.
- ❌ No capture todas las excepciones; sea específico.
