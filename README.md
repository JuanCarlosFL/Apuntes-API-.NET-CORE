# Apuntes-API-.NET-CORE

# CORS

- Es un mecanistmo que nos permite configurar restricciones sobre los recursos de nuestra API que sean requeridos por fuera del domino donde el recurso se encuentra hospedado.
- Es un estándar W3C
- Ayuda a evitar llamadas maliciosas a nuestros servicios o recursos

### Request Headers

- Origin
- Access-Control-Request-Method
- Access-Control-Request-Headers

### Response Headers

- Access-Control-Allow-Origin
- Access-Control-Allow-Credentials
- Access-Control-Expose-Headers
- Access-Control-Max-Age
- Access-Control-Allow-Methods
- Access-Control-Allow-Headers

Ejemplo.  En el método Configure de Startup.cs

```C#
app.UseCors(builder =>
 builder.WithOrigins("url")
 .WithMthods("GET", "POST", "PUT", "DELETE")
 .AllowAnyHeader())
 .Build();
```
- Es mejor usar políticas, eso iría en el método ConfigureServices de Startup.cs

```C#
services.AddCors( p=> {
 p.AddPolicy("MyPolicy", builder => {
   builder.WithOrigins("url")
  .WithMthods("GET", "POST", "PUT", "DELETE")
  .AllowAnyHeader())
  .Build();
 });
});
```

- Y el método Configure quedaría así: (de esta manera podemos tener varias políticas y aplicar una u otra en función de si estamos en desarrollo, etc..)

```C#
app.UseCors("MyPolicy");
```

- Otra forma de aplicarlo sería dejar vacío el app.UseCors() e implementarlo en el controlador que queramos, es importante que UserCors() vaya detrás de UseRouting()

```C#
using Microsoft.AspNetCore.Cors;

[EnableCors("MyPolicy")]
```

# JSON WEB TOKEN

- Es un estándar abierto basado en JSON propuesto por IETF (RPC 7519) para la creación de tokens de acceso que permiten la propagación de identidad y priviliegios o claims en inglés.
- Se usa para saber si una persona tiene acceso a un recurso o no y qué privilegios tiene sobre dicho recurso.
- Se compone de:
  - header
  - payload: información general del usuario.
  - signature: firma única que va a contener este token generado desde el servicio

- Una app primero tiene que acceder (Request, normalmente por post) a un endpoint (pj api/getoken) para obtener dicho token y luego ya podrá acceder (Request with bearer token) al controlador.

### Uso de Json Web Token
- Instalar el paquete Microsoft.AspNetCore.Authentication.JwtBearer
- En el appsettings.json crear una propiedad para la llave secreta:
  - "SecretKey": "aswdaffslkerjf^kj34*jk4@#"
- En el método Configure:
```C#
var key = Encoding.ASCII.GetBytes(Configuration.GetValue<string>("SecretKey"));

services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
}).AddJwtBearer(options =>
{
    options.TokenValidationParameters = new Microsoft.IdentityModel.Tokens.TokenValidationParameters
    {
        IssuerSigningKey = new SymmetricSecurityKey(key),
        ValidateLifetime = true,
        ValidIssuer = "",
        ValidAudience = "",
        ValidateAudience = false,
        ValidateIssuer = false,
        ValidateIssuerSigningKey = true,
    };
});
```

- Tenemos que poner una configuración para que todos nuestros controladores requieran autenticación, así evitamos que se nos olvide poner en algún contrlador el atributo [Authorize]

```C#
services.AddControllers(config =>
{
    var policy = new AuthorizationPolicyBuilder()
                        .RequireAuthenticatedUser()
                        .Build();
    config.Filters.Add(new AuthorizeFilter(policy));
});
```

- Hay que añadir el middleware app.UseAuthentication() por delante del app.UseAuthorization();
- Ahora ya podemos trabajar en el LoginUserController, se puede añadir un método al user controller pero es mejor en un controlador separado por si queremos añadirle más seguridad pj

```C#
 private readonly IConfiguration _configuration;

public JwtUserController(IConfiguration configuration)
{
    _configuration = configuration;
}

[AllowAnonymous]
[HttpGet("RequestToken")]
public JsonResult RequestToken()
{
    DateTime utcNow = DateTime.UtcNow;

    List<Claim> claims = new List<Claim>
    {
        new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
        new Claim(JwtRegisteredClaimNames.Iat, utcNow.ToString())
    };

    DateTime expiredDateTime = utcNow.AddDays(1);

    var jwtSecurityTokenHandler = new JwtSecurityTokenHandler();

    //key + credentials
    var key = Encoding.ASCII.GetBytes(_configuration.GetValue<string>("SecretKey"));
    var symmetricSecurityKey = new SymmetricSecurityKey(key);
    var signingCredentials = new SigningCredentials(symmetricSecurityKey, SecurityAlgorithms.HmacSha256);

    string token = jwtSecurityTokenHandler.WriteToken(new JwtSecurityToken(claims: claims, expires: expiredDateTime, notBefore: utcNow, signingCredentials: signingCredentials));

    return new JsonResult(new { token });
}
```

### Odata

- El protocolo de Datos Abierto (OData) o Open Data Protocoles, es un protocolo abierto que permite la creación y consumición de APIs RESTful que puedan ser consultadas e interoperables de una manera simple y estandarizada.  Microsoft inició dicho protocolo en el 2007.  Este protocolo permite obtener recursos de una manera simple, usando queries dentro de los request HTTP. Por ejemplo  https://localhost:44344/odata/students?$select=name
- Operaciones (Select, Filter, Orderby, Extend (permite hacer un Include))

### Paso 1
- Instalar el Nuget: Microsoft.Data.OData.Core

### Paso 2
- Añadir el servicio: services.AddOData(); y en services.AddControllers añadir config.EnableEndPointRouting = false.

### Paso 3 
- Añadir el middleware de las imágenes a continuación y borrar el app.UseEndPoints para dejar que OData maneje los endpoints.

![image](https://user-images.githubusercontent.com/66184823/109414017-e77f0b80-79b0-11eb-9f86-1c3aa550188e.png)

![image](https://user-images.githubusercontent.com/66184823/109414081-3fb60d80-79b1-11eb-9c90-5e48f78d5d49.png)

### Paso 4
- Añadir el atributo [EnableQuery()] al método del controlador con el que queramos usarlo.
- Y el método ahora sería public ActionResult<IQueryable<User>> Get()
- Ahora ya podemos hacer queries tal que:
 - http://localhost:5000/api/user?$select=name,userid,active&$filter=active eq true
 - http://localhost:5000/api/user?$expand=userRoles&$orderby=name
    
# Pruebas unitarias

- Se debe crear un proyecto a parte y añadirle las referencias, vamos a utilizar un proyecto con xUnit
    
    
    
    
    
    
    
    
    

