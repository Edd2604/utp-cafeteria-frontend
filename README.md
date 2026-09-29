# UTP Cafetería · Frontend

Interfaz de pedidos en línea para la cafetería de la universidad: el alumno
arma su pedido, elige la hora a la que lo va a recoger y paga; el personal lo ve
entrar en tiempo real desde el panel.

Habla con la [API](https://github.com/Edd2604/utp-cafeteria-api), un monolito
modular en NestJS.

## Las tres zonas

```
src/app/
├── (auth)/    Ingreso
├── shop/      Catálogo, búsqueda, carrito, checkout, mis pedidos, perfil
└── admin/     Productos, inventario, pedidos, ventas, usuarios
```

El `middleware.ts` las protege antes de renderizar: sin token, a la portada;
con token pero sin rol `ADMINISTRADOR`, fuera de `/admin`. La comprobación
ocurre en el borde, no dentro de la página, así que nadie llega a ver el
esqueleto de una vista que no le toca.

## Dos piezas que vale la pena mirar

### El carrito sabe qué hora es

`src/store/cart-store.ts` — Zustand con `persist`, así que el carrito sobrevive
a recargar la página. Pero además guarda la **hora de recojo**, y la valida:

```ts
const timeRange = hour >= 9 && hour < 22
const notPast   = scheduled.getTime() >= now.getTime()
return timeRange && notPast
```

Un pedido para recoger a las 3 de la madrugada, o para una hora que ya pasó, no
llega ni a enviarse. Es una regla de negocio de la cafetería —su horario—
resuelta en el cliente, antes de gastar una petición.

### Los pedidos llegan solos

Nadie recarga nada. Dos hooks escuchan por WebSocket con Pusher:

| Hook | Canal | Eventos |
|---|---|---|
| `admin/hooks/use-orders-listener` | `orders-channel` | `order:new`, `order:new-status`, `order:cancelled` |
| `shop/hooks/use-my-orders-listener` | por usuario | estado de sus propios pedidos |

El alumno ve cómo avanza su pedido y el personal ve entrar los nuevos, los dos
sin tocar el navegador.

## Stack

Next.js (App Router) · React · TypeScript · Tailwind CSS 4 ·
shadcn/ui sobre Radix UI · Zustand con persistencia · SWR · Axios ·
React Hook Form con Zod · NextAuth · Pusher · Embla Carousel · Sonner

## Estructura

```
src/
├── app/         Rutas del App Router
├── modules/     Funcionalidad por zona: admin · auth · shop · shared
├── store/       Estado del carrito
├── providers/   Contextos de la aplicación
├── lib/         Cliente HTTP y utilidades
├── config/      Configuración
└── middleware.ts
```

## Cómo levantarlo

```bash
npm install
npm run dev
```

Queda en `http://localhost:3000`. Necesita la API corriendo y un `.env` con su
URL, el secreto de NextAuth y las claves públicas de Pusher.

Otros comandos: `npm run build`, `npm start`, `npm run lint`, `npm run format`.

---

