# Open Telemetry

## Introducción

La observabilidad es un pilar fundamental en el desarrollo de aplicaciones modernas, permitiendo monitorear, diagnosticar y optimizar el rendimiento de los sistemas en tiempo real. En este contexto, OpenTelemetry se ha consolidado como un estándar abierto para la instrumentación de aplicaciones, ofreciendo una solución unificada para la recolección de métricas, trazas y logs.

## Objetivo

Este manual tiene como objetivo guiar paso a paso la integración de OpenTelemetry en un proyecto Laravel, permitiendo a los desarrolladores implementar un sistema de observabilidad robusto y escalable.

## Ventajas

- **Estándar abierto:** Compatible con múltiples proveedores de observabilidad.
- **Flexibilidad:** Permite integrarse con herramientas como Jaeger, Prometheus, entre otras.
- **Visibilidad completa:** Obtén trazas detalladas de las solicitudes y transacciones en tu aplicación.
- **Mejora continua:** Facilita la identificación de cuellos de botella y errores en tiempo real.

Con esta guía, podrás configurar OpenTelemetry en tu proyecto Laravel de manera eficiente y aprovechar al máximo sus beneficios.

## Descripción

OpenTelemetry (OTel) es un conjunto de herramientas, API y SDK de código abierto diseñado para la recolección, generación y exportación de datos de telemetría, como métricas, logs y trazas, en aplicaciones distribuidas. Su objetivo es proporcionar visibilidad completa del rendimiento y el comportamiento de los sistemas, facilitando la observabilidad en arquitecturas modernas, como microservicios y entornos en la nube, con soporte en múltiples lenguajes como Java, Python, JavaScript, Go, .NET, PHP, entre otros.

## Jaeger

Jaeger es una plataforma de código abierto para la trazabilidad distribuida, diseñada para monitorear y solucionar problemas en sistemas de microservicios. Fue desarrollada originalmente por Uber y ahora es parte de la Cloud Native Computing Foundation (CNCF).

### Pruebas locales con entornos contenerizados

Para realizar pruebas locales, puedes utilizar un entorno contenerizado. En este caso, Jaeger será la plataforma encargada de registrar y visualizar la observabilidad configurada en el proyecto.

```yaml
jaeger:
    image: jaegertracing/jaeger:2.0.0
    container_name: jaeger-server
    networks:
        - laravel_net
    ports:
        - "5778:5778"   # Agent HTTP monitoring port
        - "16686:16686" # Query UI
        - "4317:4317"   # OTLP gRPC
        - "4318:4318"   # OTLP HTTP
        - "14250:14250" # Collector gRPC
        - "14268:14268" # Collector HTTP
        - "9411:9411"   # Zipkin-compatible API
    command:
        - "--set"
        - "receivers.otlp.protocols.http.endpoint=0.0.0.0:4318"
        - "--set"
        - "receivers.otlp.protocols.grpc.endpoint=0.0.0.0:4317"
```

Construir la imagen y levantar servicios:

```bash
docker-compose up --build -d
```

## Instalación

Ejecuta el siguiente comando en la raíz de tu proyecto:

```bash
composer require keepsuit/laravel-opentelemetry
```

Luego, publica el archivo de configuración:

```bash
php artisan vendor:publish --provider="Keepsuit\LaravelOpenTelemetry\LaravelOpenTelemetryServiceProvider" --tag="opentelemetry-config"
```

## Configuración

### Exclusión de trackings innecesarios

En tu archivo de configuración, puedes exonerar ciertos trackings para evitar registros innecesarios. Dentro del array `instrumentation`, encontrarás opciones para configurar qué elementos deben ser excluidos del seguimiento.

```php
use Keepsuit\LaravelOpenTelemetry\Instrumentation\HttpServerInstrumentation;

return [
    'instrumentation' => [
        HttpServerInstrumentation::class => [
            'excluded_paths' => [
                'telescope*',
                'horizon*',
                // Otros paths a excluir...
            ],
        ],
        // Otras instrumentaciones...
    ],
    // Otras configuraciones...
];
```

Con esta configuración, puedes gestionar de manera eficiente qué rutas o componentes de tu aplicación serán monitoreados, optimizando así el rendimiento y la relevancia de los datos recopilados.

## Uso

### Desactivar la observabilidad

Para poder activar o desactivar Open Telemetry, actualmente la variable `OTEL_SDK_DISABLED=false` en el .env no es escuchada, la forma más práctica es dirigirse a `bootstrap/app.php` y antes del `return` del código, insertar el siguiente fragmento:

```php
// Desactiva completamente OpenTelemetry desde la raíz
putenv('OTEL_SDK_DISABLED=true');
```