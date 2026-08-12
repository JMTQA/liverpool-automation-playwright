1.	¿Qué aspectos de este proceso no automatizarías y por qué?

    No automatizaría los objetos o componentes dinámicos de CSS que cambian con frecuencia y el costo de mantenimiento de pruebas seria alto, ya que los productos cambian contante mente, así como sus precios por rebaja.

2.	Si Liverpool añadiera un CAPTCHA al proceso de búsqueda, ¿cómo lo gestionarías en tu conjunto de pruebas?

  Seria complicado va en contra de las mejores prácticas de QA. Ya que el CAPTCHA es la seguridad de internet que sirve para saber si quien usa una página web es una persona real y no un programa de computadora o robot, por el cual es complicado.

  En su caso se podría hacer semi automatizado, el cual se deja un lapso de tiempo detenida la ejecución de prueba mientras manualmente realiza una persona el CAPTCHA y posteriormente seguir con el proceso automatizado.


3.	¿Qué riesgos de inestabilidad existen en esta prueba y cómo los mitigaste?

  En primer punto seria la carga asíncrona de elementos tardan mucho los renderizados dinámicos de las tarjetas de producto en la carga y puede gener fallos por tiempo de espera muy prolongados.
  Mitigación: Se implementó el uso de selectores robustos genérico en combinación con esperas explícitas de estado nativo de Playwright, en lugar de hardcoded sleeps.
  
  Como segundo sería los cambios dinámicos en la estructura de clases CSS: Liverpool utiliza clases compiladas (ej. `text-body-base`). 
  Así como el anidado de elemento en frame que hace difícil la interacción con objetos. 
  
  Mitigación:  Se utilizaron atributos estáticos de prueba como `data-testid` y ejes XPath basados en contenido textual (`.//span[text()='Texto']`).
  Para los Frame de debe realizar entrar al marco padre y luego al hijo antes de buscar el elemento para poder realizar interacción con el o ver si hay algún cambio cuando carga nueva pantallas emergentes.


4.	Si tuvieras que añadir esto a la canalización de integración continua de un equipo que ejecuta más de 50 conjuntos de pruebas, ¿qué cambiarías?

  Para integrar esta suite a un pipeline masivo de integración continua, realizaría los siguientes cambios:
  
  Ejecución Paralela: Distribuir las pruebas utilizando Playwright Sharding o Selenium Grid/Docker o bien Jenkins para reducir los tiempos globales de ejecución.
  
  Estrategia de Ejecución por Capas: Utilizar etiquetas de Testing para ejecutar los flujos de (Alta (P1), Media (P2) y Baja (P3).
  
  Dividir los conjuntos de pruebas: Prioridad y dividir las pruebas en (Smoke test, regresión, integración, funcionales y No Funcionales).
