# Gestor de Gastos Compartidos (tipo Splitwise)

## ¿Qué es esta aplicación?

Esta aplicación permite registrar y repartir gastos entre varias personas. La idea es que un grupo de amigos, compañeros de piso, de viaje o de evento, pueda apuntar quién paga qué y entre quiénes se reparte.
Aquí tenéis el ejemplo de la aplicación 

Por ejemplo:
> Ana paga una cena de 60€ entre ella, Juan y Luis.  
> La app registra ese gasto y automáticamente calcula que Juan y Luis le deben 20€ cada uno a Ana.

La app debe mostrar los gastos, quién ha pagado, entre cuántas personas se reparte, y calcular las **deudas pendientes** entre ellos.

---

## Tecnologías obligatorias

- **Frontend**: React + React Router
- **Backend**: Node.js + Express
- **Base de datos**: MongoDB + Mongoose

---

## Organización del proyecto

### Backend

El backend se encargará de:

- Gestionar los datos de **usuarios** y **gastos**
- Calcular las deudas según los gastos registrados
- Exponer una **API REST** con rutas para cada operación CRUD

Rutas necesarias:
| Método | Ruta            | Descripción                        |
|--------|------------------|------------------------------------|
| GET    | /expenses        | Listar todos los gastos            |
| GET    | /expenses/:id    | Ver un gasto por su ID             |
| POST   | /expenses        | Crear un nuevo gasto               |
| PUT    | /expenses/:id    | Editar un gasto                    |
| DELETE | /expenses/:id    | Borrar un gasto                    |
| GET    | /users           | Listar usuarios                    |
| POST   | /users           | Crear un usuario nuevo             |

---

### 📦 Base de datos (MongoDB)

Se usarán **dos colecciones**: `users` y `expenses`.

#### Modelo: `User`

Este modelo representa a las personas que forman parte del grupo de gastos.

```js
{
  _id: ObjectId,
  name: String,
  email: String
}
```
## ¿Por qué es importante el modelo de usuario?

Los usuarios son la base para poder registrar los gastos.  
Cada gasto tiene un `paidBy` (quién lo pagó) y una lista de `participants` (los que lo comparten).  
Por tanto, **debe haber usuarios creados antes de registrar ningún gasto**.

---

### Ejemplo práctico inicial

Crea 3 usuarios desde el frontend o con peticiones manuales:

```json
{ "name": "Ana", "email": "ana@email.com" }
{ "name": "Juan", "email": "juan@email.com" }
{ "name": "Luis", "email": "luis@email.com" }
```
Con esos tres usuarios se podrán añadir gastos en los que participen.

#### Modelo: `Expense`

Este modelo representa cada gasto que se ha hecho.

```js
{
  _id: ObjectId,
  title: String,
  amount: Number,
  paidBy: ObjectId,         // Usuario que pagó
  participants: [ObjectId], // Usuarios entre los que se reparte
  date: Date,
  description: String
}
```
#### Ejemplo de gasto

```js
{
  "title": "Cena en restaurante",
  "amount": 60,
  "paidBy": "ana_id",
  "participants": ["ana_id", "juan_id", "luis_id"],
  "description": "Cena del viernes por la noche",
  "date": "2025-07-22T00:00:00.000Z"
}
```
Este gasto significa que Ana ha pagado 60€ que se reparten entre los tres → cada uno paga 20€,
pero como Ana ya ha pagado 60€, Juan y Luis le deben 20€ cada uno.

### Frontend (React)
El frontend debe mostrar y permitir navegar entre las diferentes secciones de la aplicación.
Usaremos React Router para manejar las vistas.

### Páginas obligatorias
| Ruta                 | Descripción                                               |
| -------------------- | --------------------------------------------------------- |
| `/`                  | Página principal con resumen de deudas                    |
| `/expenses`          | Listado completo de gastos registrados                    |
| `/expenses/:id`      | Detalle de un gasto                                       |
| `/expenses/new`      | Formulario para crear un nuevo gasto                      |
| `/expenses/:id/edit` | Editar un gasto existente                                 |
| `/users`             | Lista de usuarios (opcional, útil para ver participantes) |

### Lógica del reparto (cálculo de deudas)
Cuando se registra un gasto:
- Se toma el importe total
- Se divide entre el número de participantes
- Se calcula cuánto le deben al que pagó

#### Ejemplo real

```js
- // Gasto:
title: "Supermercado"
amount: 90
paidBy: Ana
participants: Ana, Juan, Luis

// Cálculo:
90€ ÷ 3 = 30€ cada uno

// Resultado:
- Juan debe 30€ a Ana
- Luis debe 30€ a Ana
```

La lógica de estos cálculos se puede hacer:
- En el backend (por ejemplo, para las rutas de resumen o cálculos agregados)
- O en el frontend (al mostrar el reparto en tiempo real)

⚠️ No hace falta crear una colección de "deudas":
Las deudas pueden calcularse dinámicamente a partir de los gastos registrados.
