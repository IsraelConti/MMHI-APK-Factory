# MMHI APK Factory

Herramienta de compilación y control de calidad para APK Android de proyectos creados en **Cursor, Expo, Android Studio o Google AI Studio (modo Android nativo)**.

## Qué hace hoy (v0.1, base técnica)
1. Lee el código, detecta si es un proyecto **Expo o Android** y busca evidencias de funciones (cámara, OCR, Excel, almacenamiento, compartir, localización y micrófono).
2. Compara esas evidencias con un contrato `mmhi-functions.json` que mantiene el equipo. Si faltan funciones declaradas, detiene la compilación.
3. Compila mediante `:app:assembleRelease`, no `assembleDebug`. **Bloquea `expo-dev-client`** para no volver a entregar una aplicación que solicite URL de Metro.
4. Comprueba estructura del APK, firma, paquete y, en Expo, el bundle JavaScript integrado.
5. Guarda un artefacto descargable con `mmhi-release.apk`, `mmhi-functions-report.json` y `mmhi-validation.json`.

**Importante:** la detección en código no prueba el funcionamiento real. Cámara, OCR, permisos, guardado y exportación deben someterse a pruebas en emulador o teléfono antes de considerar una versión terminada.

## Enlace a la fábrica
[Ejecutar/inspeccionar GitHub Actions](../../actions/workflows/build-apk.yml)

## Integración en Cursor
Copia [esta regla Cursor](.cursor/rules/mmhi-apk-factory.mdc) al mismo directorio de tu proyecto para que el agente revise y documente todas las funciones antes de compilar.

Copia [este ejemplo de contrato](examples/mmhi-functions.json) a la raíz de cada proyecto con nombre `mmhi-functions.json`. Completa los requisitos reales; no supongas que todos los proyectos usan OCR o cámara.

En el **repositorio GitHub de cada aplicación**, crea `.github/workflows/compilar-mmhi.yml` con:

```yaml
name: Crear APK MMHI
on:
  workflow_dispatch:
    inputs:
      project_dir:
        description: Ruta relativa del proyecto (ej. apps/mmhi-mobile)
        type: string
        default: "."
jobs:
  release:
    uses: IsraelConti/MMHI-APK-Factory/.github/workflows/build-apk.yml@main
    with:
      project_dir: ${{ inputs.project_dir }}
```

Después entra en la pestaña **Actions** del repositorio de la aplicación, abre **Crear APK MMHI** y ejecuta el flujo. Descarga `mmhi-release.apk` desde el artefacto final de esa ejecución. El flujo no publica en Play Store.

## Integración en Google AI Studio
- Si se creó una aplicación **Android nativa (Kotlin/Jetpack Compose)**, exporta o sincroniza el proyecto con GitHub y añade el flujo anterior.
- Si se creó una aplicación **web React/Node**, **NO es automáticamente una APK nativa**. Hay que implementar una capa Android y adaptar cámara, permisos, ficheros, almacenamiento y servicios. La fábrica rechaza proyectos solo web para evitar falsos éxitos.
- Un prompt de Gemini debe generar también `mmhi-functions.json` y pruebas funcionales que representen las pantallas, permisos y funciones originales.

## Limitaciones y seguridad
- **Estado:** el workflow está registrado en GitHub, pero todavía no se ha validado una ejecución completa con un proyecto Android integrado.
- El repositorio de la fábrica NO contiene las aplicaciones ni necesita sus claves.
- Los APK sin firma Release se firman con una **clave temporal de pruebas** generada en cada ejecución. Para actualizar versiones sin desinstalar y para Play Store se requiere configurar **firma estable y protegida**. No usar la firma temporal para distribuir versiones permanentes.
- Los artefactos GitHub Actions tienen caducidad, no son un enlace permanente. Para compartir públicamente: publicar después el APK validado en una Release o Drive.
- Las funciones con API en nube requieren sus credenciales y servicios reales; compilar un APK no los sustituye.

## Regla de entrega
**NO ENTREGAR** una APK si pide URL, si falla la firma, si faltan funcionalidades obligatorias o si el código JS de Expo no va incluido. La prueba física de cámara/OCR debe figurar explícitamente como realizada o pendiente.
