# Mantecados · Toma de pedidos

App de mostrador. Carga de pedidos, envío de comanda por WhatsApp y caja diaria
con registro en Firestore, compartido entre todos los dispositivos del local.

## Archivos

| Archivo | Qué es |
|---|---|
| `index.html` | La app entera. Editás `firebaseConfig`, `CONFIG` y `CARTA` arriba del `<script>`. |
| `firestore.rules` | Reglas de seguridad. Sin esto la base queda abierta. |
| `firebase.json` | Config de Hosting. `public` en `"."` e ignora `firebase.exe`. |
| `mascota.png` | **Falta.** Guardá acá la foto de perfil del IG (el pickle). |

## Puesta en marcha

### 1. Proyecto en Firebase
En la consola: **Build → Firestore Database → Crear** (modo producción) y
**Build → Hosting → Comenzar** (esto último antes del primer deploy, para evitar
el error de *hosting target with no site name*).

### 2. Cuentas del local
**Build → Authentication → Sign-in method → Correo/contraseña → Habilitar.**
Después, en la pestaña **Users → Agregar usuario**, creá una cuenta por local o
por puesto. No hay registro público desde la app: las cuentas se crean a mano
desde la consola, que es lo correcto para uso interno.

### 3. Config
Copiá las claves de **Configuración del proyecto → Tus apps → SDK** y pegalas en
`firebaseConfig`. Cargá también el número de WhatsApp de cocina en `CONFIG`
(formato `54` + `9` + área sin 0 + número sin 15).

### 4. Reglas
Pegá el contenido de `firestore.rules` en **Firestore → Reglas → Publicar**,
o dejá que suban solas con el deploy (ya están declaradas en `firebase.json`).

### 5. Deploy
1. Bajá `firebase.exe` de https://firebase.tools/bin/win/instant/latest y ponelo
   en esta carpeta.
2. Creá `.firebaserc` con el ID del proyecto:
   ```json
   { "projects": { "default": "TU-PROJECT-ID" } }
   ```
3. Doble clic en `firebase.exe`, y en esa consola: `firebase login` y después
   `firebase deploy`.

`firebase.exe` ya está en la lista de `ignore`, así que el plan Spark no va a
rechazar el deploy por subir un ejecutable.

## Probarlo antes de subirlo

No alcanza con abrir `index.html` con doble clic: usa módulos ES, que el
navegador bloquea desde `file://`. Levantá un servidor local — `firebase serve`
desde la consola de `firebase.exe`, o `python -m http.server` en la carpeta.

## Cómo queda la base

```
pedidos/{autoId}     numero, fecha, dia, cliente, nota, items[],
                     total, pago, anulado, creadoPor, creadoEn
contadores/{dia}     valor          ← contadores/2026-09-08 → { valor: 37 }
```

El número de pedido y el pedido se escriben en la misma transacción, así que dos
dispositivos tomando pedidos al mismo tiempo nunca generan el mismo número.
El contador arranca en 1 cada día.

El precio de cada producto queda **copiado** dentro del pedido, no referenciado a
`CARTA`. Si mañana subís precios, el histórico sigue mostrando lo que se cobró.

## Cambiar la carta

Editá el array `CARTA`. Podés cambiar nombres y precios cuando quieras; lo único
que no hay que tocar es el `id` de cada producto, porque es lo que quedó grabado
en los pedidos viejos.

Si en algún momento el local quiere editar precios sin tocar el código, el paso
siguiente es mover `CARTA` a una colección de Firestore y agregar una pantalla de
administración. Mientras los cambios de precio pasen por vos, no hace falta.
