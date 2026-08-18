# Matriz de autorizacion - Boveda

Documento entregable de la Practica 2. Una fila por operacion. La columna
`authz.ts` apunta a la linea real de `src/lib/authz.ts` que implementa la regla:
si no podes apuntar a una linea concreta, esa regla esta regada por el codigo
en vez de vivir en la politica, y eso es un hallazgo que hay que anotar.

| Operacion | ANALISTA | APROBADOR | AUDITOR | Condicion ABAC | authz.ts |
|---|---|---|---|---|---|
| Ver solicitud | Solo su sucursal | Solo su sucursal | Todas | s.sucursalId === actor.sucursalId, salvo AUDITOR | L17 |
| Crear solicitud | Si | No | No | actor.rol === 'ANALISTA' (Para que tenga sentido los roles porque aprobador solo aprueba y auditor solo revisa) | L26 |
| Resolver solicitud | No | Si | No | misma sucursal + estado PENDIENTE + no es quien la creo | L37 |
| Ver bitacora | No | No | Si | actor.rol === 'AUDITOR' | L54 |

## Los agujeros que justifican cada linea

Una linea por agujero: que se rompe si esa regla no existe.

- Agujero A1: Analista lee transferencias de otra sucursal distinta a la suya. Se fugaron las transferencias y eso afectaría al banco porque una persona está viendo datos sensibles no autorizados.
- Agujero A2: El auditor no puede ver las transferencias y sí debería para poder revisar. A veces una regla que parece inofensiva puede afectar otras caracteristicas si no se toma todo en cuenta.
- Agujero B1: El auditor puede crear transferencias, y no debería poder porque el es quien audita.
- Agujero B2: El aprobador puede crear transferencias y no debería para poder tener una segregación de funciones. 

- Resolver / rol: 
- Resolver / sucursal: 
- Resolver / estado: 
- Resolver / doble control: 
- Ver / sucursal: 
- Ver / excepcion del auditor: 
- Crear / roles excluidos: 

## Evidencia de ataques fallidos

Para cada ataque: que se intento, que respondio la app, y **que quedo en la base**
(el estado sin cambiar es lo que prueba la defensa, no el mensaje de error).

1. 
2. 
3. 
4. 
5. 