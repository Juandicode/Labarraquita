# La Barraquita — Sitio estático migrado a AWS (S3 + CloudFront)

Sitio web estático para una ferreteria, migrado desde Netlify a una arquitectura de referencia en AWS como proyecto de portfolio para roles de Cloud/DevOps.

**🔗 Sitio en producción:** `https://d1bsgcjseh0k7h.cloudfront.net`

> Nota: el sitio fue desarrollado con asistencia de IA (Claude) para el frontend. El foco de este proyecto es la arquitectura de hosting cloud, seguridad y despliegue — no el diseño del sitio en sí.

---

## Contexto

Este sitio corría originalmente en Netlify (donde sigue disponible como hosting de producción real para clientes de mi micro-empresa de desarrollo web). Elegí migrar **este proyecto propio** (sin riesgo para terceros) a AWS para aprender de forma práctica la arquitectura que se usa en contextos empresariales, donde Netlify/Vercel no siempre son opción por integración con otros sistemas internos de la empresa.

**Aclaración honesta:** para un sitio estático simple como este, Netlify sigue siendo la opción más eficiente (deploy en segundos, gratis, sin mantenimiento de infraestructura). Esta migración es un ejercicio deliberado para demostrar comprensión de arquitectura cloud, no una recomendación de "mejor solución" para este caso de uso puntual.

---

## Arquitectura

```
Usuario → CloudFront (CDN + HTTPS) → OAC → S3 bucket (privado)
```

- **Amazon S3**: almacena los archivos estáticos del sitio (HTML, CSS, JS, imágenes). Bucket completamente privado, sin acceso público directo.
- **Amazon CloudFront**: CDN que sirve el contenido públicamente, con HTTPS incluido por default (dominio `.cloudfront.net`).
- **Origin Access Control (OAC)**: mecanismo que autoriza *únicamente* a esta distribución de CloudFront a leer del bucket S3. Ningún otro request, ni siquiera con la URL directa de S3, puede acceder al contenido.

---

## Decisiones técnicas y por qué

| Decisión | Alternativa descartada | Motivo |
|---|---|---|
| Bucket S3 privado + OAC | Bucket público con website hosting habilitado | Principio de mínimo privilegio: el bucket no necesita ser accesible directamente, solo a través de CloudFront. Reduce superficie de ataque. |
| ACLs deshabilitadas | ACLs habilitadas | Acceso gestionado solo por políticas (bucket policy), más simple de auditar y es la práctica recomendada actual por AWS. |
| Sin AWS WAF | Habilitar WAF | Costo fijo mensual (~US$5-6 + por tráfico) no se justifica para un sitio de bajo volumen en fase de portfolio/desarrollo. Documentado como conocido pero no implementado. |
| Dominio default de CloudFront (`.cloudfront.net`) | Dominio propio vía Route 53 + ACM | Route 53 tiene costo (compra del dominio + US$0.50/mes por hosted zone). CloudFront ya da HTTPS gratis en su dominio default, suficiente para validar la arquitectura sin gasto. Queda documentado como próximo paso posible. |
| Sin versionado de bucket | Versionado habilitado | Sitio simple sin necesidad actual de rollback de archivos históricos. Se puede habilitar sin downtime si se necesita más adelante. |
| Cifrado SSE-S3 (default) | SSE-KMS / DSSE-KMS | SSE-S3 es gratuito y suficiente para contenido no sensible como este. KMS agrega costo sin beneficio real acá. |

---

## Troubleshooting real durante el proyecto

**Problema:** al acceder a la URL de CloudFront por primera vez, la respuesta era `403 AccessDenied` en vez de mostrar el sitio.

**Causa:** CloudFront no tenía configurado un *default root object*. Al entrar a la raíz del dominio (`/`), CloudFront no sabía que debía servir `index.html` — no hay redirección automática a ese archivo por default.

**Solución:** configurar `index.html` como "Default root object" en los ajustes generales (`Settings`) de la distribución de CloudFront. Tras la propagación (~2-3 minutos), el sitio cargó correctamente.

---

## Costos

- S3: dentro de free tier (almacenamiento mínimo, pocos GB)
- CloudFront: dentro de free tier (bajo volumen de requests)
- WAF: no habilitado — $0
- Route 53 / dominio propio: no implementado en esta fase — $0
- **Total actual: $0/mes** (dentro de los límites del free tier de AWS)

---

## Próximos pasos posibles

- [ ] Dominio propio vía Route 53 + certificado ACM
- [ ] Habilitar versionado de bucket
- [ ] Configurar CloudFront Functions para redirects/headers de seguridad adicionales
- [ ] Pipeline de deploy automático (relacionado con el proyecto de CI/CD del portfolio)

---

## Stack

`AWS S3` `AWS CloudFront` `Origin Access Control (OAC)` `HTML/CSS/JS`

---

*Proyecto 1 de 4 de un portfolio orientado a roles de Cloud/DevOps (AWS/Azure), construido sobre productos reales de mi micro-emprendimiento de desarrollo web.*
