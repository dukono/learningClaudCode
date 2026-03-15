# 8. /add-dir - Agregar Directorios

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 8. /add-dir
[⬆️ Top](#8-add-dir---agregar-directorios)

**¿Qué hace?**
Agrega un directorio adicional al workspace de Claude Code. Por defecto, Claude solo tiene acceso al directorio desde donde fue iniciado. `/add-dir` expande ese acceso a directorios adicionales, útil para monorepos o proyectos multi-repositorio.

**Funcionamiento interno:**

```
Sin /add-dir:
claude iniciado en /proyectos/frontend/

Claude puede acceder a:
  ✓ /proyectos/frontend/**       ← directorio de trabajo
  ✗ /proyectos/backend/          ← NO accesible
  ✗ /proyectos/shared/           ← NO accesible

Con /add-dir /proyectos/backend:
Claude puede acceder a:
  ✓ /proyectos/frontend/**       ← directorio original
  ✓ /proyectos/backend/**        ← ahora accesible
  ✗ /proyectos/shared/           ← aún NO accesible
```

**Uso práctico:**

```bash
/add-dir /ruta/al/directorio

# Ejemplos:
/add-dir ../backend                    # Directorio hermano relativo
/add-dir /home/user/proyectos/shared   # Ruta absoluta
/add-dir ~/libs/mi-libreria            # Con tilde
```

**Casos de uso reales:**

```bash
# Caso 1: Monorepo - trabajar con frontend y backend juntos
# Iniciado en: /monorepo/packages/frontend
/add-dir ../backend
/add-dir ../shared
"Revisa cómo el frontend consume los tipos de ../shared y si hay inconsistencias"

# Caso 2: Librería compartida entre proyectos
# Iniciado en: /mi-app
/add-dir ~/libs/design-system
"Actualiza los imports de componentes a la nueva versión del design-system"

# Caso 3: Configuración compartida
/add-dir /etc/mi-proyecto
"Lee la configuración de producción y compárala con la de desarrollo"
```

**Mejores prácticas:**

```
✅ Usar rutas absolutas para mayor claridad
✅ Añadir solo los directorios que realmente necesitas
✅ Útil para monorepos donde los paquetes se referencian entre sí

❌ No agregar directorios innecesariamente (aumenta tokens si Claude lee mucho)
❌ No agregar directorios con datos sensibles si no es necesario
```
