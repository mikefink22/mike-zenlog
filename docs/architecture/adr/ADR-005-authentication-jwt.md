# ADR-005: Autenticación JWT Propia con Modo Local y Soporte Multi-Cliente (Web + Mobile)

- **Estado**: Aceptado
- **Fecha**: 2026-11-01
- **Contexto**: El usuario usará la agenda inicialmente en su máquina local sin querer perder tiempo logueándose cada vez que abre el navegador, pero en la Fase 2 (Web Online) y Fase 3 (App Mobile) necesita proteger su sesión de forma segura.

## Decisión

Implementar **Autenticación basada en JWT (JSON Web Tokens) gestionada por nuestro propio Backend (`apps/api`)** con dos modos de operación controlados por variables de entorno (`.env.example`):

1. **Modo Local Rápido (`AUTH_MODE=local_single_user`)**:
   - Habilitado únicamente cuando `NODE_ENV=development`.
   - Un usuario por defecto (`local@agenda.dev`) se crea automáticamente en el seed local y el middleware de autenticación inyecta su sesión sin pedir contraseña, permitiendo usar la agenda al instante.
2. **Modo Autenticado (`AUTH_MODE=jwt`)**:
   - Obligatorio en `NODE_ENV=production` (y activable en local para probar el flujo de login).
   - Contraseñas hasheadas con `scrypt` / `argon2` o `bcryptjs`.
   - Emisión de `accessToken` (JWT firmado con `JWT_SECRET`) compatible tanto con `HttpOnly Cookie` (Web) como con header `Authorization: Bearer <token>` (futura App Mobile en React Native).

## Consecuencias

- **Positivas**: Independiente de proveedores de pago; funciona 100% offline en local; sirve por igual a la Web y a la futura App Mobile.
- **Seguridad**: `JWT_SECRET` nunca se sube al repositorio; se documenta un placeholder en `.env.example` y el hook de seguridad impide que el agente lea el `.env` real del usuario.
