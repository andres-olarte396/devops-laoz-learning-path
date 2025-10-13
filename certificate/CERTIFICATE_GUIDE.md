# Generador de Certificados - Guía de Uso

Este archivo HTML genera certificados personalizables usando parámetros de URL. Todos los textos e imágenes son completamente parametrizables.

## Parámetros Disponibles

### Parámetros Básicos

| Parámetro | Descripción               | Valor por Defecto         |
| --------- | ------------------------- | ------------------------- |
| `nombre`  | Nombre del participante   | "Nombre del Participante" |
| `curso`   | Nombre del curso/programa | "Curso de DevOps"         |
| `fecha`   | Fecha y lugar de emisión  | "Lima, [fecha actual]"    |
| `codigo`  | Código del certificado    | "CERT-[timestamp]"        |

### Parámetros de Contenido

| Parámetro        | Descripción                       | Valor por Defecto                                             |
| ---------------- | --------------------------------- | ------------------------------------------------------------- |
| `titulo`         | Título principal del certificado  | "Certificado de Participación"                                |
| `organizacion`   | Nombre de la organización emisora | "DevOps Learning Path"                                        |
| `subtitulo`      | Texto bajo el título              | "[organizacion] concede el presente a:"                       |
| `textoCompletar` | Texto antes del nombre del curso  | "Por completar con éxito el programa de estudios denominado:" |
| `descripcion`    | Descripción detallada del curso   | Texto por defecto sobre DevOps                                |

### Parámetros de Firmas

| Parámetro      | Descripción                          | Valor por Defecto      |
| -------------- | ------------------------------------ | ---------------------- |
| `firma1Nombre` | Nombre del primer firmante           | "Instructor Principal" |
| `firma1Cargo`  | Cargo del primer firmante            | "Líder Técnico"        |
| `firma1Img`    | URL de la imagen de la primera firma | URL por defecto        |
| `firma2Nombre` | Nombre del segundo firmante          | "Director Académico"   |
| `firma2Cargo`  | Cargo del segundo firmante           | "PhD, DevOps Expert"   |
| `firma2Img`    | URL de la imagen de la segunda firma | URL por defecto        |

### Parámetros de Imágenes

| Parámetro        | Descripción                                          | Valor por Defecto  |
| ---------------- | ---------------------------------------------------- | ------------------ |
| `logo`           | URL del logo (esquina superior izquierda)            | Icono de nube      |
| `sello`          | URL del sello (esquina superior derecha)             | Medalla dorada     |
| `fondo`          | URL de la imagen de fondo del certificado            | "certificate.webp" |
| `overlayOpacity` | Opacidad del overlay blanco sobre el fondo (0.0-1.0) | "0.85"             |

### Parámetros de Tema

| Parámetro      | Descripción                                     | Valor por Defecto                    |
| -------------- | ----------------------------------------------- | ------------------------------------ |
| `tema`         | Tema del certificado: "light", "dark", "auto"   | "light"                              |
| `primaryColor` | Color primario personalizado (hex, rgb, nombre) | "#007bff" (light) / "#4dabf7" (dark) |
| `bgColor`      | Color de fondo personalizado                    | "#fff" (light) / "#1a1a1a" (dark)    |
| `textColor`    | Color del texto personalizado                   | "#333" (light) / "#e0e0e0" (dark)    |

## Ejemplos de Uso

### Ejemplo Básico

```
certificate.html?nombre=Juan%20Pérez&curso=DevOps%20Fundamentals
```

### Ejemplo con Tema Oscuro

```url
certificate.html?nombre=Ana%20Martínez&curso=DevOps%20Expert&tema=dark
```

### Ejemplo con Colores Personalizados

```url
certificate.html?nombre=Carlos%20López&tema=dark&primaryColor=%23ff6b6b&textColor=%23f8f9fa
```

### Ejemplo con Imagen de Fondo Personalizada

```url
certificate.html?nombre=Ana%20Martínez&curso=DevOps%20Expert&fondo=https://example.com/mi-fondo.jpg&overlayOpacity=0.7
```

### Ejemplo Completo con Fondo

```
certificate.html?nombre=María%20García%20López&curso=DevOps%20Advanced%20-%20Fase%202&organizacion=Tech%20Academy&titulo=Certificado%20de%20Excelencia&descripcion=Ha%20demostrado%20dominio%20avanzado%20en%20Docker,%20Kubernetes,%20Terraform%20y%20CI/CD&fecha=Barcelona,%2020%20de%20octubre%20de%202025&codigo=DVP-ADV-2025-042&firma1Nombre=Dr.%20Ana%20Martínez&firma1Cargo=CTO&firma2Nombre=Ing.%20Roberto%20Silva&firma2Cargo=DevOps%20Architect
```

### Ejemplo para Fase 1 del Roadmap

```
certificate.html?nombre=Carlos%20Rodríguez&curso=Fundamentos%20de%20DevOps%20-%20Fase%201&organizacion=DevOps%20Learning%20Path&titulo=Certificado%20de%20Finalización%20-%20Fase%201&descripcion=Ha%20completado%20exitosamente%20los%20fundamentos%20de%20DevOps%20incluyendo%20conceptos%20básicos,%20Git,%20CI/CD,%20automatización%20y%20mejores%20prácticas&fecha=Lima,%2015%20de%20octubre%20de%202025&codigo=DLP-F1-2025-001
```

## Codificación de URL

Recuerda que los espacios y caracteres especiales deben estar codificados en URL:

- Espacio → `%20`
- Ñ → `%C3%B1`
- Á → `%C3%81`
- É → `%C3%89`
- Í → `%C3%8D`
- Ó → `%C3%93`
- Ú → `%C3%9A`

## Funcionalidades

### Descarga

- **Imagen PNG**: Descarga el certificado como imagen de alta resolución
- **PDF**: Descarga el certificado como archivo PDF en formato horizontal

### Responsive

- El certificado se adapta automáticamente a diferentes tamaños de pantalla
- Optimizado para impresión y visualización digital

## Integración con el Roadmap

Este generador puede integrarse fácilmente con el sistema de evaluación del roadmap DevOps:

### Para cada fase completada

```javascript
// Generar URL de certificado para fase completada
function generarCertificadoFase(nombreEstudiante, numeroFase) {
  const baseUrl = "certificate.html";
  const params = new URLSearchParams({
    nombre: nombreEstudiante,
    curso: `Fundamentos de DevOps - Fase ${numeroFase}`,
    organizacion: "DevOps Learning Path",
    titulo: `Certificado de Finalización - Fase ${numeroFase}`,
    fecha: `Lima, ${new Date().toLocaleDateString("es-ES")}`,
    codigo: `DLP-F${numeroFase}-${new Date().getFullYear()}-${Math.random()
      .toString(36)
      .substr(2, 6)
      .toUpperCase()}`,
  });

  return baseUrl + "?" + params.toString();
}
```

### URLs Sugeridas por Fase

#### Fase 0 - Terminal

```
certificate.html?curso=Dominio%20de%20la%20Terminal%20-%20Fase%200&descripcion=Ha%20dominado%20los%20fundamentos%20de%20la%20línea%20de%20comandos,%20scripting%20básico%20y%20herramientas%20CLI%20esenciales%20para%20DevOps
```

#### Fase 1 - Fundamentos DevOps

```
certificate.html?curso=Fundamentos%20de%20DevOps%20-%20Fase%201&descripcion=Ha%20completado%20los%20conceptos%20fundamentales%20de%20DevOps,%20Git,%20CI/CD,%20automatización%20e%20infraestructura%20como%20código
```

#### Fase 2 - CI/CD

```
certificate.html?curso=Automatización%20y%20CI/CD%20-%20Fase%202&descripcion=Ha%20demostrado%20competencia%20en%20pipelines%20CI/CD,%20Jenkins,%20GitHub%20Actions%20y%20estrategias%20de%20deployment%20avanzadas
```

## Personalización Avanzada

Para personalizar aún más el certificado, puedes modificar:

1. **Estilos CSS**: Cambiar colores, fuentes, layout
2. **Marcas de agua**: Modificar el patrón de fondo
3. **Validación**: Agregar QR codes o enlaces de verificación
4. **Idiomas**: Soportar múltiples idiomas via parámetros

### Consejos para Imágenes de Fondo

**Resolución recomendada:** 1100x800 píxeles o superior
**Formato:** WebP, JPG, PNG
**Opacidad del overlay:**

- 0.9-1.0 para fondos muy claros o text-friendly
- 0.7-0.8 para fondos con textura moderada
- 0.5-0.6 para fondos muy detallados o oscuros

**Ejemplos de overlayOpacity:**

```url
# Fondo muy visible (overlay sutil)
certificate.html?fondo=mi-fondo-claro.jpg&overlayOpacity=0.3

# Fondo balanceado (overlay moderado)
certificate.html?fondo=mi-fondo-textura.jpg&overlayOpacity=0.7

# Fondo apenas visible (overlay fuerte)
certificate.html?fondo=mi-fondo-complejo.jpg&overlayOpacity=0.9
```

## Troubleshooting

### Problemas Comunes

**Las imágenes no cargan**

- Verifica que las URLs de `logo`, `sello`, `firma1Img`, `firma2Img` sean accesibles
- Usa URLs HTTPS para evitar problemas de contenido mixto

**Caracteres especiales no se muestran**

- Asegúrate de codificar correctamente los parámetros URL
- Usa herramientas online de codificación URL

**El PDF/imagen se ve cortado**

- El certificado está optimizado para resolución estándar
- Para certificados muy largos, considera ajustar el CSS

## Recursos

- [Codificador URL Online](https://www.urlencoder.org/)
- [Generador de QR Codes](https://qr.io/)
- [Free Logo/Icon Resources](https://www.flaticon.com/)
