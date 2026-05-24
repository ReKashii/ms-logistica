### 📂 Estructura del Código Creado                                                                                                                                                               
                                                                                                                                                                                                    
  Los archivos se han distribuido en el directorio  ms-logistica/src/main/java/cl/bookpointchile/logistica  conforme al patrón CSR:                                                                 
                                                                                                                                                                                                    
    ms-logistica/                                                                                                                                                                                   
    ├── pom.xml                                     # Dependencias del proyecto (Lombok, Validation, JPA)                                                                                           
    └── src/                                                                                                                                                                                        
        └── main/                                                                                                                                                                                   
            ├── java/                                                                                                                                                                               
            │   └── cl/                                                                                                                                                                             
            │       └── bookpointchile/                                                                                                                                                             
            │           └── logistica/                                                                                                                                                              
            │               ├── config/                                                                                                                                                             
            │               │   └── DataInitializer.java  # Sembrador de rutas de distribución y envíos base                                                                                        
            │               ├── controller/                                                                                                                                                         
            │               │   └── LogisticaController.java # Endpoints de creación y actualización de estados                                                                                     
            │               ├── dto/                                                                                                                                                                
            │               │   ├── CrearEnvioRequestDTO.java     # JSR 380 para registrar despachos                                                                                                
            │               │   ├── ActualizarEstadoRequestDTO.java # JSR 380 para control de transiciones                                                                                          
            │               │   └── EnvioResponseDTO.java         # DTO con datos consolidados de ruta y despacho                                                                                   
            │               ├── exception/                                                                                                                                                          
            │               │   ├── ResourceNotFoundException.java                                                                                                                                  
            │               │   ├── TransicionEstadoInvalidaException.java # HTTP 400 Bad Request                                                                                                   
            │               │   ├── ErrorResponse.java            # Formato estándar de errores REST                                                                                                
            │               │   └── GlobalExceptionHandler.java    # Interceptor global @RestControllerAdvice                                                                                       
            │               ├── model/                                                                                                                                                              
            │               │   ├── EstadoEnvio.java              # Enum (PENDIENTE, EN_RUTA, ENTREGADO, DEVUELTO)                                                                                  
            │               │   ├── RutaDistribucion.java         # Entidad de rutas geográficas                                                                                                    
            │               │   └── Envio.java                    # Entidad transaccional de seguimiento                                                                                            
            │               ├── repository/                                                                                                                                                         
            │               │   ├── RutaDistribucionRepository.java                                                                                                                                 
            │               │   └── EnvioRepository.java          # Consultas transaccionales de despachos                                                                                          
            │               ├── service/                                                                                                                                                            
            │               │   ├── LogisticaService.java                                                                                                                                           
            │               │   └── LogisticaServiceImpl.java     # Máquina de estados y ruteador de direcciones                                                                                    
            │               └── LogisticaApplication.java         # Bootstrap de Spring Boot                                                                                                        
            └── resources/                                                                                                                                                                          
                └── application.properties                  # Configuración de base de datos MySQL y puerto 8085                                                                                    
  ──────                                                                                                                                                                                            
  ### 🛠️ Decisiones de Desarrollo y Diseño                                                                                                                                                            
                                                                                                                                                                                                    
  1. Máquina de Estados Estricta (Lógica Logística):                                                                                                                                                
      • En LogisticaServiceImpl.java se ha implementado una máquina de estados estricta en el método  actualizarEstado . Esto previene transiciones de estado imposibles o ilógicas en la vida real.
Por       
      ejemplo:                                                                                                                                                                                      
          • Si intentas cambiar de  PENDIENTE  a  ENTREGADO  o  DEVUELTO  de forma directa sin pasar por  EN_RUTA , se denegará la petición, se levantará una alerta  log.warn  en consola y se     
          lanzará la excepción  TransicionEstadoInvalidaException  (HTTP 400).                                                                                                                      
          • Los estados  ENTREGADO  y  DEVUELTO  se marcan como estados finales inalterables.                                                                                                       
                                                                                                                                                                                                    
  2. Ruteador Logístico Inteligente:                                                                                                                                                                
      • Si al registrar un despacho mediante  CrearEnvioRequestDTO  no se especifica un  rutaId , el servicio analiza sintácticamente la dirección de destino. Si detecta la comuna de "Hualpén" o  
      "Talcahuano", asociará automáticamente la ruta optimizada correspondiente y asignará al courier a cargo de esa zona, simulando un sistema de despacho inteligente en producción.              
  3. Garantía de Unicidad en Despachos:                                                                                                                                                             
      • Envio.java define un índice único a nivel base de datos ( UniqueConstraint ) sobre la columna  venta_id . Esto asegura físicamente que una misma venta no pueda tener dos despachos     
      activos por error del sistema.                                                                                                                                                                
  4. Validaciones Beans Estrictas (JSR 380):                                                                                                                                                        
      • En CrearEnvioRequestDTO.java se valida de forma sintáctica que la dirección no esté vacía ( @NotBlank ) y que el identificador de venta esté presente ( @NotNull ).                                   
  5. Sembrado de Rutas de Prueba (Data Seeder):                                                                                                                                                     
      • He programado DataInitializer.java. En el primer arranque, registrará de forma automática las rutas base que parten desde la "Bodega Central Concepción" hacia Hualpén, Talcahuano y una ruta    
      general de la Región del Bío Bío, además de insertar despachos de prueba en distintos estados para verificar inmediatamente las transiciones de negocio.                                      
                                                                                                                                                                                                    
  ──────                                                                                                                                                                                            
  ### ⚙️ Propiedades del Entorno                                                                                                                                                                     
                                                                                                                                                                                                    
  En application.properties se han ajustado los siguientes parámetros:                                                                                                                                     
                                                                                                                                                                                                    
  • Puerto:  server.port=8085  (independiente de ms-ventas  8081 , ms-inventario  8082 , ms-usuarios  8083  y ms-catalogo  8084 ).                                                                  
  • Base de datos: MySQL esquema  bookpoint_logistica  (con  createDatabaseIfNotExist=true ).                                                                                                       
  • Logging: Activado a nivel  INFO  para el rastreo logístico de despachos y auditoría de transportes en consola.                                                                                  
  ──────                                                                                                                                                                                            
  ### 🔍 Endpoints REST Expuestos                                                                                                                                                                   
                                                                                                                                                                                                    
  •  POST /api/logistica/envios : Registra un nuevo despacho a domicilio asociado a una venta (dispara ruteador inteligente y  @Valid).                                                            
  •  PUT /api/logistica/envios/{id}/estado : Actualiza el estado de seguimiento del pedido (evalúa la máquina de estados logística).                                                                
  •  GET /api/logistica/envios/venta/{ventaId} : Permite al cliente web o al Jefe de Sucursal rastrear en tiempo real el estado de despacho de su compra mediante el ID de la venta.
