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
  options.DefaultAuthenticationScheme = JwtBearerDefaults.AuthenticationScheme;
  options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
}).AddJwtBearer(options => {
  options.TokenValidationParameters = new Microsoft.IdentityModel.Tokens.TokenVa
  {
    IssuerSigningKey = new SymetricSecurityKey(key),
    ValidateLifetime = true,
    ValidIssuer = "",
    ValidAudience = "",
    ValidateAudience = false,
    ValidateIssuer = false,
    ValidateIssuerSigningKey = true
  }
});
```

- Hay que añadir el middleware app.UseAuthentication() por delante del app.UseAuthorization();
- Ahora ya podemos trabajar en el LoginUserController, se puede añadir un método al user controller pero es mejor en un controlador separado por si queremos añadirle más seguridad pj

```C#
private readonly IConfiguration _configuration;

public LoginUserController(IConfiguration configuration)
{
  _configuration = configuration
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
  }
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    

