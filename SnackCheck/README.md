## SnackCheck

SnackCheck es una automatización desarrollada en n8n que permite comprobar si un producto es saludable a partir de su código de barras.

El sistema consulta los datos nutricionales del producto en Open Food Facts, calcula una clasificación mediante los valores de azúcar, sal y grasa, y la combina con el Nutri-Score.

Finalmente, Groq genera una explicación breve y fácil de entender para el usuario.

## Funcionamiento

El flujo recibe un código de barras mediante un Webhook.

Después:

Comprueba que se haya enviado un código de barras.
Comprueba que el código contenga únicamente números.
Consulta el producto en Open Food Facts.
Comprueba si el producto existe.
Obtiene sus datos nutricionales.
Comprueba que existan los datos necesarios de azúcar, sal y grasa.
Calcula la clasificación mediante el semáforo nutricional.
Comprueba el Nutri-Score.
Combina ambas clasificaciones utilizando el peor resultado.
Groq genera el mensaje final para el usuario.
El Webhook devuelve la respuesta.

## API utilizada

SnackCheck utiliza la API de Open Food Facts para obtener la información del producto.

La consulta se realiza mediante:

```text
GET https://world.openfoodfacts.org/api/v2/product/{barcode}.json?fields=product_name,brands,nutriscore_grade,nutriments
```

Los datos utilizados son:

- Nombre del producto.
- Marca.
- Nutri-Score.
- Azúcar por 100 g.
- Sal por 100 g.
- Grasa por 100 g.
- Calorías por 100 g.

No es necesario utilizar una API Key para consultar Open Food Facts.

## Manejo de errores

El flujo controla diferentes situaciones para evitar que la automatización falle:

- Si no se proporciona un código de barras, devuelve un mensaje indicando que falta.
- Si el código contiene caracteres que no son numéricos, devuelve un mensaje de código no válido.
- Si el producto no se encuentra, informa al usuario.
- Si faltan datos nutricionales necesarios, indica que no hay información suficiente para realizar la clasificación.

En todos estos casos se devuelve una respuesta clara al usuario en lugar de detener el flujo con un error.

## Clasificación

La clasificación utiliza dos métodos independientes:

### Semáforo nutricional

Se comprueban los valores por cada 100 g:

- Azúcar: alto si supera 22,5 g.
- Sal: alto si supera 1,5 g.
- Grasa: alto si supera 17,5 g.

Según el número de nutrientes altos:

- 0 → saludable
- 1 → moderado
- 2 o 3 → no saludable

### Nutri-Score

- A o B → saludable
- C → moderado
- D o E → no saludable

La clasificación final utiliza el peor resultado entre el semáforo y el Nutri-Score. Si no existe Nutri-Score, se utiliza únicamente el semáforo.

## Generación de la respuesta

Una vez calculada la clasificación, se utiliza un modelo de Groq para generar el mensaje final.

Groq no realiza la clasificación del producto. Recibe los datos ya calculados y redacta una respuesta breve y fácil de entender para el usuario.

La respuesta incluye la clasificación y los principales datos nutricionales del producto.

## Pruebas

El workflow se puede probar enviando una petición POST al Webhook con un código de barras.

Ejemplo:

```json
{
  "barcode": "3017620422003"
}
```

También se deben comprobar los siguientes casos:

- Petición sin código de barras.
- Código de barras con caracteres no numéricos.
- Producto inexistente.
- Producto con datos nutricionales insuficientes.
- Producto con una clasificación saludable.
- Producto con una clasificación no saludable.

## Requisitos

Para ejecutar el workflow se necesita:

- n8n.
- Una credencial de Groq configurada.
- Acceso a Internet para consultar Open Food Facts.

Open Food Facts no requiere una API Key para esta consulta.
