# Informe del Experimento: Creación de Infraestructura con AWS CDK

Este repositorio contiene el informe detallado sobre el experimento realizado para crear una infraestructura en la nube utilizando el AWS Cloud Development Kit (CDK) de AWS. El objetivo de este experimento es desplegar un stack básico de AWS con una función Lambda simple.

## Requisitos Previos

Antes de comenzar con el experimento, asegúrate de tener configurado lo siguiente:

1. **Cuenta de AWS**: Necesitas una cuenta activa en AWS para poder desplegar los recursos en este caso se tiene una cuenta de vocarona que da acceso y por ende se realizan pasos distintos.
2. **AWS CLI**: Debes tener instalada y configurada la AWS CLI en tu máquina.
3. **Java 8 o superior**: AWS CDK para Java requiere tener instalado el JDK de Java (Java 8 o superior).
4. **Maven**: AWS CDK para Java usa Maven como herramienta de construcción, por lo que necesitarás tener Maven instalado.
5. **AWS CDK**: El AWS CDK debe estar instalado globalmente en tu máquina. Puedes instalarlo con el siguiente comando:
    ```  
    npm install -g aws-cdk  
    ```
6. **Instalación de Dependencias**: Una vez clonado el repositorio, navega hasta el directorio del proyecto e instala las dependencias utilizando el siguiente comando:  
    ```  
    mvn clean install  
    ```

## Pasos para Crear la Infraestructura con AWS CDK

A continuación, se detallan los pasos para seguir el tutorial "Hello World" de AWS CDK y crear una infraestructura básica en la nube.

### 1. Inicializar el Proyecto

Primero, creamos un nuevo proyecto de AWS CDK. Este comando genera una estructura básica de directorios y archivos para empezar:

```  
cdk init app --language=java  
```

Este comando crea una carpeta con el nombre de tu proyecto y una serie de archivos necesarios para el desarrollo con CDK.

### 2. Cambios de acceso por IAM

Para este tutorial, debido a que por vocarona no se tiene acceso a IAM para realizar la ejecución del tutorial se deben hacer varios cambios en el paso de bootstrap

1. Se debe generar un yml del template con el siguiente comando
```  
cdk bootstrap --show-template > bootstrap-template.yaml
```
2. En el archivo "bootstrap-template.yaml" se debe comentar todos el area de "Resources" de accesos a Roles
3. En el archivo "bootstrap-template.yaml" se debe reemplazar "FileAssetsBucketEncryptionKey" en la propiedad "action" 
```  
Fn::Sub: ${FilePublishingRole.Arn}
```
Deberia quedar así
```  
Fn::Sub: "*"
```
4. Y se ejecuta el deploy para el uso de este boostrap

```  
cdk bootstrap --template bootstrap-template.yaml
```
### 3. Crear la Función Lambda

La siguiente etapa del tutorial consiste en crear una función Lambda básica que responderá a las solicitudes de API Gateway.

Dentro del archivo `HelloCdkStack`, agregamos el siguiente código para definir una función Lambda simple:

```
// Import Lambda function
import software.amazon.awscdk.services.lambda.Code;
import software.amazon.awscdk.services.lambda.Function;
import software.amazon.awscdk.services.lambda.Runtime;
// Import Lambda function URL
import software.amazon.awscdk.services.lambda.FunctionUrl;
import software.amazon.awscdk.services.lambda.FunctionUrlAuthType;
import software.amazon.awscdk.services.lambda.FunctionUrlOptions;
// Import CfnOutput
import software.amazon.awscdk.CfnOutput;
import software.amazon.awscdk.services.iam.Role;
          // Define the Lambda function resource
          Function myFunction = Function.Builder.create(this, "HelloWorldFunction")
                .runtime(Runtime.NODEJS_20_X) // Provide any supported Node.js runtime
                .handler("index.handler")
                .code(Code.fromInline(
                        "exports.handler = async function(event) {"
                        + " return {"
                        + " statusCode: 200,"
                        + " body: JSON.stringify('Hello World!')"
                        + " };"
                        + "};"))
                .role(Role.fromRoleArn(this, "LabRole", "arn:aws:iam::706852032701:role/LabRole"))
                .build();

        // Define the Lambda function URL resource
        FunctionUrl myFunctionUrl = myFunction.addFunctionUrl(FunctionUrlOptions.builder()
                .authType(FunctionUrlAuthType.NONE)
                .build());

        // Define a CloudFormation output for your URL
        CfnOutput.Builder.create(this, "myFunctionUrlOutput")
                .value(myFunctionUrl.getUrl())
                .build();
```

Este código crea una función Lambda en Node.js y un API Gateway que expone la Lambda como un servicio HTTP.

### 4. Desplegar la Infraestructura

Para desplegar la infraestructura definida con CDK, usamos los siguientes comandos:

1. **Compilar el código java**:
   ```  
   mvn compile -q 
   ```

2. **Realizar un "synth" del stack**:
   ```  
   cdk synth  
   ```
   ![image](https://github.com/user-attachments/assets/505fd150-8955-4513-ac25-87e73035006c)

3. **Realizar un "deploy" del stack**:

 Debido a que usamos un rol especifico el despliegue cambia por este comando  
   ```  
   cdk deploy -r arn:aws:iam::706852032701:role/LabRole
   ```

Este comando creará los recursos en tu cuenta de AWS, como la función Lambda. Al finalizar, recibirás una URL pública para acceder al servicio creado.
![image](https://github.com/user-attachments/assets/0c25a5c5-8ad6-42c6-b305-c0f7c272fd4f)

### 5. Probar la Aplicación

Una vez desplegada la infraestructura, puedes probar la aplicación accediendo a la URL que se muestra en la salida del comando `cdk deploy`. Simplemente abre tu navegador y visita la URL proporcionada para ver la respuesta de la función Lambda:

```  
{  
  "message": "Hello, CDK!"  
}  
```
![image](https://github.com/user-attachments/assets/a3c9f046-0324-4f0e-81f6-440b5dbbe8f1)

## Detalles del Experimento

Por ultimo si se quiere ver como quedo la infraestructura del cdk nos diigimos a cloud information en aws y podemos ver creado el stack

![image](https://github.com/user-attachments/assets/5d4857c6-7b19-451e-9635-d31ede3a31fa)

![image](https://github.com/user-attachments/assets/971a4a46-4a8a-4f16-abfe-beebb49e7919)


## Conclusión

AWS CDK simplifica la creación y gestión de recursos en la nube al permitir definir la infraestructura como código. En este experimento, se ha logrado crear una aplicación simple con AWS Lambda, lo que demuestra el potencial de AWS CDK para manejar infraestructuras más complejas de manera eficiente y reproducible.

## Recursos

- [Documentación del tutorial Hello World](https://docs.aws.amazon.com/cdk/v2/guide/hello_world.html)
- [Documentación del tutorial bootstrap](https://docs.aws.amazon.com/cdk/v2/guide/bootstrapping-env.html)

