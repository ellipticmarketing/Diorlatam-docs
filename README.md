# Integración de login con Diorlatina

## Flujo de información
Antes de considerar cualquier otra información, es necesario considerar el flujo de procesos e información:
1.	Cliente entra a diorlatina.com 
2.	Cliente visita la sección de descargas y hace click en el ícono del DAM.
3.	Diorlatina envía una solicitud POST con la data del usuario. 
4.	El DAM devuelve una URL de login única para el usuario.
5.	Diorlatina redirige al usuario a esta URL.
6.	El usuario entra al DAM.

## Generación de URL de sesión.
Para solicitar una URL de inicio de sesión, se debe enviar una solicitud POST con los siguientes datos del usuario:

- name: string
- email: string
- countries: array []
    - id: string
    - name: string
    - clients: array []
        - id: string
        - name: string
        - points_of_sale: array []
            - id: string
            - name: string

Ejemplo:
```json
{
    "name": "John Doe",
    "email": "john.doe@test.test",
    "countries": [
        {
            "id": "7",
            "name": "Bolivia",
            "clients": [
                {
                    "id": "45",
                    "name": "Aromas",
                    "points_of_sale": [
                        {
                            "id": "42",
                            "name": "Aromas Casa Zonia Comercio"
                        },
                        {
                            "id": "43",
                            "name": "Aromas Perfumería Glamour"
                        }
                    ]
                }
            ]
        }
    ]
}
```

El DAM responderá con un código de estado 200 y un objeto JSON con la URL de inicio de sesión.:
```json
{
    "url": "https://www.diorlatam.com/login/3?expires=1708630057&signature=194dc4744ba7b8339c65f14907957652397492"
}
```