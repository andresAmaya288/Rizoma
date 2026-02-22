# Sistema de Reserva de Camisetas — Guía de Setup

## 📋 Resumen

El formulario de camisetas en la web está conectado a **Google Sheets** mediante **Google Apps Script**. Cuando un usuario realiza una reserva, los datos se guardan automáticamente en tu hoja de cálculo sin que tengas que hacer nada adicional.

## 🚀 Paso 1: Crear Google Sheet

1. Ve a [Google Sheets](https://sheets.google.com)
2. Crea una nueva hoja de cálculo llamada "Rizoma - Reservas de Camisetas"
3. En la primera fila, añade estos encabezados:
   ```
   Timestamp | Model | Size | Quantity | Name | Email | Phone
   ```

## 🔧 Paso 2: Crear Google Apps Script

1. En tu hoja de cálculo, ve a **Extensiones → Apps Script**
2. Reemplaza el código con esto:

```javascript
const SHEET_ID = "1GDRpgmOMjn3AxIrTxEdOioHHKvAwRrj17px282fssjg"; // Reemplaza con tu Sheet ID
const SHEET_NAME = "Datos Reservas";

function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName(SHEET_NAME);
    
    // Agregar fila con datos
    sheet.appendRow([
      data.timestamp,
      data.model,
      data.size,
      data.quantity,
      data.name,
      data.email,
      data.phone
    ]);
    
    return ContentService.createTextOutput(JSON.stringify({ status: "success" }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({ status: "error", message: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Guarda el script

## 🔑 Paso 3: Obtener tu Sheet ID

1. Abre tu hoja de cálculo
2. En la URL (`https://docs.google.com/spreadsheets/d/SHEET_ID/edit`), copia la parte entre `/d/` y `/edit`
3. Ejemplo: `https://docs.google.com/spreadsheets/d/1a2b3c4d5e6f/edit` → ID es `1a2b3c4d5e6f`

## 📤 Paso 4: Deploy del Apps Script

1. En el editor de Apps Script, haz clic en **Deploy** (botón azul arriba a la derecha)
2. Selecciona **New deployment**
3. Tipo: **Web app**
4. Ejecutar como: Tu cuenta
5. Acceso: **Anyone** (para que funcione sin autenticación)
6. Haz clic en **Deploy**
7. Se abrirá un modal con tu **Deployment URL** — **cópiala**

## 🌐 Paso 5: Actualizar la URL en index.html

1. Abre `index.html` en tu editor
2. Busca esta línea (~línea 105):
   ```javascript
   const SCRIPT_URL = 'https://script.google.com/macros/d/YOUR_SCRIPT_ID/usercallback';
   ```
3. Reemplaza con tu URL de deployment completa:
   ```javascript
   const SCRIPT_URL = 'https://script.google.com/macros/d/AKfycbX...../usercallback';
   ```
4. Guarda

## ✅ Paso 6: Prueba

1. Abre tu sitio en navegador
2. Ve a la sección "Reserva de Camisetas"
3. Rellena el formulario y haz clic en "Confirmar reserva"
4. Deberías ver el mensaje de confirmación
5. Verifica que los datos aparezcan en tu Google Sheet

## 📊 Consultar Pedidos

Los datos se guardan automáticamente en tu Google Sheet. Puedes:

- **Ver en tiempo real** todos los pedidos
- **Filtrar** por modelo, talla, etc.
- **Crear gráficos** y resúmenes
- **Exportar a CSV/Excel** para procesarlos

### Resumen por Talla y Modelo

Crea una nueva pestaña en tu hoja con fórmulas tipo:
```
=COUNTIFS(Sheet1!C:C,"M",Sheet1!B:B,"modelo-1")
```

Esto te da el total de talla M del modelo 1.

## 🛡️ Prevención de Duplicados

El sistema actual **permite duplicados** (si alguien recarga y reenvía el formulario).

**Para evitarlo**, puedes:

### Opción A: Almacenar hashes en memoria (cliente)
```javascript
// En el script, antes del addEventListener
const submittedForms = new Set();

tshirtForm.addEventListener('submit', async (e) => {
  // ... resto del código
  const formHash = btoa(JSON.stringify(formData)); // crear hash
  if (submittedForms.has(formHash)) {
    alert('Esta reserva ya fue registrada');
    return;
  }
  submittedForms.add(formHash);
  // ... continuar con envío
});
```

**Limitación:** Se resetea si recarga la página.

### Opción B: Validar en Google Sheets
Agrega una columna "Enviado" y formula que evite duplicados:
```javascript
// En Apps Script
const lastSubmission = sheet.getRange(sheet.getLastRow(), 1).getValue();
if (lastSubmission === data.timestamp) {
  return ContentService.createTextOutput(JSON.stringify({ status: "duplicate" }));
}
```

**Recomendación:** Opción A es suficiente para un evento.

## 📧 Notificaciones por Email (Opcional)

Puedes añadir en Apps Script:

```javascript
MailApp.sendEmail(data.email, "Reserva confirmada - Rizoma", 
  `Tu reserva: ${data.quantity} camiseta(s) - ${data.model} - Talla ${data.size}`);
```

## ❓ Troubleshooting

- **"Error en la reserva"** → Revisa la URL del Apps Script en `index.html`
- **No aparecen datos** → Verifica que el Apps Script tiene permisos en la hoja
- **Error 403** → Asegúrate de que el Apps Script esté publicado como "Web app - Anyone"

---

**¿Necesitas ayuda?** Puedo implementar validación de duplicados o notificaciones por email.
