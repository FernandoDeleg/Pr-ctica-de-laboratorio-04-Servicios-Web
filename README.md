

PRÁCTICA DE LABORATORIO
CARRERA: COMPUTACION	ASIGNATURA: PLATAFORMAS WEB
NRO. PRÁCTICA:	4	TÍTULO PRÁCTICA: Desarrollo de APIs usando RestFul.
OBJETIVOS
•	Diseña y desarrolla modelos de software en diferentes niveles de abstracción y modelos de datos a nivel transaccional y analítico con entornos de desarrollo actuales














INSTRUCCIONES	
Con base a la Practica de laboratorio 03 - EJB - JSF – JPA se pide aumentar las funcionalidades de la aplicación JEE usando Web Services Restful. Los servicios web deben permitir:

•	Gestión de cuentas de usuario de los Clientes de la distribuidora
o	Iniciar Sesión con base a un usuario y contraseña
o	Registrar cuenta del cliente con base al número de cédula
o	Modificar datos de la cuenta y personales del cliente
o	Anular cuenta del cliente. (eliminado lógico)
•	Gestión de Pedidos
o	Listar productos del catálogo organizados por categorías con base a la selección de una bodega.
o	Enviar la solicitud de un pedido a la distribuidora
o	Revisar el estado de los pedidos del cliente.

Por último, se deben aplicar los cambios correspondientes al modelo de negocio de la aplicación JEE de la practica 03 para que realice la:

•	Gestión de pedidos. - Los estados de los pedidos debe ser “Enviado”, “Receptado”, “En proceso”, “En camino”, “Finalizado”. Una vez que el pedido ha sido Receptado por la distribuidora se genera la factura para el cliente.
•	Gestión de cuentas de clientes. - Los clientes de la distribuidora pueden crear una cuenta para poder generar pedidos. Si el cliente no está registrado en la base de datos de la distribuidora no podrá registrarse ni iniciar sesión. El registro se debe realizar con base a la cédula más los datos correspondientes a la cuenta del usuario.












ACTIVIDADES POR DESARROLLAR
1.	Crear un repositorio en GitHub con el nombre “Práctica de laboratorio 04: Servicios Web”









 


2. Desarrollar una aplicación con tecnología JEE para gestionar pedidos a una distribuidora de productos para el hogar a través de servicios web.

Para empezar con los servicios primero creamos un nuevo paquete y dentro de este paquete estará los servicios para la gestión de pedidos y de usuarios.
 

Dentro de UsuarioRest tenemos el siguiente código:
Para realizar el registro del cliente:

@Path("/usuario")
public class UsuarioRest {

    @EJB
    private UsuarioFacade usuarioFacade;
    private Jsonb jsonb;

    @POST
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    public Response createUsuario(String jsonUsuario) {
        System.out.println("entrando al metodo de crear usuario.............................. " );
        jsonb = JsonbBuilder.create();
        System.out.println("Usuario en registro " + jsonUsuario);

        try {
            Usuario newUsuaio = jsonb.fromJson(jsonUsuario, Usuario.class);
            newUsuaio.setCambioPassword(true);
            newUsuaio.setActivo(true);
            usuarioFacade.create(newUsuaio);

            return Response.ok().entity(newUsuaio)
                    .header("Access-Control-Allow-Origin", "*")
                    .header("Access-Control-Allow-Methods", "GET, POST, DELETE, PUT")
                    .header("Access-Control-Allow-Headers", "origin, content-type, accept, authorization")
                    .header("Access-Control-Allow-Credentials", "true")
                    .allow("OPTIONS").build();

        } catch (Exception e) {

            return Response.status(500).entity("Usuario no creado: " + e)
                    .header("Access-Control-Allow-Origin", "*")
                    .header("Access-Control-Allow-Methods", "GET, POST, DELETE, PUT")
                    .header("Access-Control-Allow-Headers", "origin, content-type, accept, authorization")
                    .header("Access-Control-Allow-Credentials", "true")
                    .allow("OPTIONS").build();

        }
    }
 
Luego de ello debemos probar el servicio en el postman de la siguiente manera:

 

Al Darle Send en el postman nos aparecerá lo siguiente: 
 
Al aparecernos esto se puede decir que el servicio está funcionando correctamente, pero para confirmar podemos revisar en la base de datos de MySQL y verificar que el nuevo cliente este en la base.

Antes:

 

Después:

 

Actualizar Usuario:
    @PUT
    @Path("/{usuarioID}")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.TEXT_PLAIN)
    public Response editDireccion(@PathParam("usuarioID") String usuarioID, String jsonLocalidad) {
        //System.out.println("usuario cedula "+id);
        if (usuarioID != null) {
            jsonb = JsonbBuilder.create();
            Usuario usuario = usuarioFacade.find(usuarioID);
            if (usuario != null) {
                try {
                    usuarioFacade.edit(jsonb.fromJson(jsonLocalidad, Usuario.class));
                    return Response.ok().entity("Usuario actualizado").header("Access-Control-Allow-Origin", "*").build();
                } catch (Exception e) {
                    return Response.status(500).entity("Error al actualizar: " + e)
                            .header("Access-Control-Allow-Origin", "*")
                            .header("Access-Control-Allow-Methods", "GET, POST, DELETE, PUT")
                            .header("Access-Control-Allow-Headers", "origin, content-type, accept, authorization")
                            .header("Access-Control-Allow-Credentials", "true")
                            .allow("OPTIONS").build();
                }
            } else {
                return Response.status(Response.Status.NOT_FOUND).entity("Usuario no encontrado")
                        .header("Access-Control-Allow-Origin", "*")
                        .header("Access-Control-Allow-Methods", "GET, POST, DELETE, PUT")
                        .header("Access-Control-Allow-Headers", "origin, content-type, accept, authorization")
                        .header("Access-Control-Allow-Credentials", "true")
                        .allow("OPTIONS").build();
            }

        } else {
            return Response.status(Response.Status.NOT_FOUND).entity("Datos insuficientes")
                    .header("Access-Control-Allow-Origin", "*")
                    .header("Access-Control-Allow-Methods", "GET, POST, DELETE, PUT")
                    .header("Access-Control-Allow-Headers", "origin, content-type, accept, authorization")
                    .header("Access-Control-Allow-Credentials", "true")
                    .allow("OPTIONS").build();
        }
    }


















Prueba con el postman:
 
Cambiamos algunos valores para realizar la actualización y probamos:
 
Verificamos en la base de datos que los datos estén actualizados:
 

Código para Login:
@POST
    @Path("/login")
    @Consumes(MediaType.APPLICATION_FORM_URLENCODED)
    @Produces(MediaType.TEXT_PLAIN)
    public Response loginUsuario(@FormParam("correo") String correo, @FormParam("password") String password) {
    	
        if (correo != null && password != null) {
            jsonb = JsonbBuilder.create();
            Usuario usuario = usuarioFacade.finByEmailAndPass(correo, password);

            System.out.println("Json Usuario "+jsonb.toJson(usuario));
            if (usuario != null) {

                if (usuario.getRol().equals("cliente")) {
                    try {
                        HttpSession session = Session.getSession();
                        session.setAttribute("token", session.getId());
                        session.setAttribute("usuario", usuario);

                        //System.out.println("Session iniciada con " + session.getId());
                    } catch (Exception e) {
                        System.out.println("Error en la sesion: " + e);
                    }

                    return Response.ok(jsonb.toJson(usuario)).header("Access-Control-Allow-Origin", "*").build();

                } else {
                    return Response.status(Response.Status.NOT_FOUND).entity("Usuario no existe").build();
                }

            } else {
                return Response.status(Response.Status.NOT_FOUND).entity("Usuario no existe").build();
            }

        } else {
            return Response.status(Response.Status.NOT_FOUND).entity("Datos insuficientes").build();
        }
    }

Prueba en postman:
 

         Eliminado Lógico:

    @DELETE
    @Path("/{usuarioID}")
    @Produces(MediaType.APPLICATION_JSON)
    public Response deleteDireccion(@PathParam("usuarioID") String usuarioID) {
        //System.out.println("usuario cedula "+id);
        if (usuarioID != null) {
            jsonb = JsonbBuilder.create();
            Usuario usuario = usuarioFacade.find(usuarioID);

            if (usuario != null) {
                try {
                    usuario.setActivo(false);
                    usuarioFacade.edit(usuario);
                    return Response.ok().entity("Usuario eliminado")
                            .header("Access-Control-Allow-Origin", "*")
                            .header("Access-Control-Allow-Methods", "GET, POST, DELETE, PUT")
                            .header("Access-Control-Allow-Headers", "origin, content-type, accept, authorization")
                            .header("Access-Control-Allow-Credentials", "true")
                            .allow("OPTIONS").build();

                } catch (Exception e) {
                    return Response.status(500).entity("Error al eliminar: " + e)
                            .header("Access-Control-Allow-Origin", "*")
                            .header("Access-Control-Allow-Methods", "GET, POST, DELETE, PUT")
                            .header("Access-Control-Allow-Headers", "origin, content-type, accept, authorization")
                            .header("Access-Control-Allow-Credentials", "true")
                            .allow("OPTIONS").build();
                }
            } else {
                return Response.status(Response.Status.NOT_FOUND).entity("Usuario no encontrado")
                        .header("Access-Control-Allow-Origin", "*")
                        .header("Access-Control-Allow-Methods", "GET, POST, DELETE, PUT")
                        .header("Access-Control-Allow-Headers", "origin, content-type, accept, authorization")
                        .header("Access-Control-Allow-Credentials", "true")
                        .allow("OPTIONS").build();
            }

        } else {
            return Response.status(Response.Status.NOT_FOUND).entity("Datos insuficientes")
                    .header("Access-Control-Allow-Origin", "*")
                    .header("Access-Control-Allow-Methods", "GET, POST, DELETE, PUT")
                    .header("Access-Control-Allow-Headers", "origin, content-type, accept, authorization")
                    .header("Access-Control-Allow-Credentials", "true")
                    .allow("OPTIONS").build();
        }
    }

Prueba con el Postman:

 

En este ejemplo solamente debemos añadir el numero de la cédula de la persona poder eliminar.
Y nos aparecerá la siguiente en el postman:
 
En la base de datos se mostrará lo siguiente:
 
Como podemos ver en la columna de ACTIVO tenemos que 1 es que la cuenta esta activa y 0 para cuando una cuenta no esta activa al realizar el eliminado lógico en la base de datos aún se verá reflejado el cliente, pero su estado pasa a estar inactivo es decir 0.


Gestión de Pedidos:
Listar productos del catálogo organizados por categorías con base a la selección de una bodega.

Para ello utilizamos el siguiente código:

@POST
    @Path ("/bodega")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getBodega(@FormParam("bodega")Integer id) {
        System.out.println("Entrando al metodo................................ ");
        Jsonb jsonb = JsonbBuilder.create();
        Categoria cate = categoriaFacade.find(id);

        List<Producto> pro = productoFacade.findByBodega(id);
       

        System.out.println("CAtegoria ........." + pro);
        return Response.ok(jsonb.toJson(pro)).header("Access-Control-Allow-Origin", "*").build();
    }

Prueba con Postman:
 
Esto funciona listándonos los productos que tiene una bodega de acuerdo con su código.
 


Estados de la factura:
Se realiza con el siguiente código:
@GET
	@Path("/estado/{cedula}")
	@Produces(MediaType.APPLICATION_JSON)
	public Response listaFacestados(@PathParam("cedula") String usuarioID) {
		if (usuarioID != null) {
			Usuario usuario = usuarioFacade.find(usuarioID);

			if (usuario != null) {
				Jsonb jsonb = JsonbBuilder.create();
				List<FacturaCabecera> listaFacturas = facturaCabeceraFacade.findAll();

				for (int i = 0; i < listaFacturas.size(); i++) {
					if (listaFacturas.get(i).getUsuario().getCedula().equals(usuario.getCedula())) {
						estadosFacturasCliente.add(listaFacturas.get(i).getUsuario().getNombre());
						estadosFacturasCliente.add(listaFacturas.get(i).getCodigo());
						estadosFacturasCliente.add(listaFacturas.get(i).getEstado());
						estadosFacturasCliente.add(listaFacturas.get(i).getFecha());
						estadosFacturasCliente.add(listaFacturas.get(i).getTotal());

					}

				}
				System.out.println(estadosFacturasCliente.toString());
				return Response.ok(jsonb.toJson(estadosFacturasCliente)).header("Access-Control-Allow-Origin", "*")
						.build();
			} else {
				return Response.status(404).entity("El cliente no ha realizado pedidos").build();
			}
		} else {
			return Response.status(404).entity("El cliente no ha realizado pedidos").build();
		}
	}




Pruebas con Postman:
 

Realizar pedido:
@POST
	@Path("/pedido/{usuarioID}")
	@Consumes(MediaType.APPLICATION_JSON)
	@Produces(MediaType.TEXT_PLAIN)
	public Response addDireccion(@PathParam("usuarioID") String usuarioID, String jsonFactura) {

		if (usuarioID != null) {
			Usuario usuario = usuarioFacade.find(usuarioID);

			if (usuario != null) {
				try {
					List<Localidad> listaLOcalidades = new ArrayList<Localidad>();
					List<Producto> listaProductos = new ArrayList<Producto>();
					List<FacturaDetalle> listaDetalles = new ArrayList<FacturaDetalle>();
					// FacturaCabecera fc = new FacturaCabecera();
					// fc.setFecha(LocalDate.now());

					JSONObject obj = new JSONObject(jsonFactura);
					listaLOcalidades = localidadFacade.listaLocalidad();
					producto = productoFacade.ByName(String.valueOf(obj.get("productos")));
					listaProductos = productoFacade.findByName(String.valueOf(obj.get("productos")));

					for (int i = 0; i < listaLOcalidades.size(); i++) {
						if (listaLOcalidades.get(i).getUsuario() != null
								&& listaLOcalidades.get(i).getUsuario().getCedula().equals(usuario.getCedula())) {
							localidad = listaLOcalidades.get(i);
							listaDetalles.add(facturaDetalle = new FacturaDetalle(
									Integer.parseInt(String.valueOf(obj.get("cantidad"))), producto));

							facturaCabecera = new FacturaCabecera(LocalDate.now(), facturaDetalle.getSubtotal(),
									usuario, listaDetalles, localidad);
							facturaCabecera.calcularSubTotal();
							facturaCabecera.calcularTotal();
							facturaCabeceraFacade.create(facturaCabecera);
							System.out.println(facturaCabecera.toString());
						}
					}

					return Response.ok().entity("Pedido creado").build();
				} catch (Exception e) {
					return Response.status(500).entity("Usuario no creado: " + e).build();
				}
			} else {
				return Response.status(Response.Status.NOT_FOUND).entity("Usuario no encontrado").build();
			}
		} else {
			return Response.status(Response.Status.NOT_FOUND).entity("Datos insuficientes").build();
		}
	}
Prueba con Postman:
 

Gestión de pedidos. - Los estados de los pedidos debe ser “Enviado”, “Receptado”, “En proceso”, “En camino”, “Finalizado”. Una vez que el pedido ha sido Receptado por la distribuidora se genera la factura para
el cliente.


Estos cambios se verían reflejados en la parte de la interfaz por lo que nos queda de la siguiente manera:














 
Como se pide que ahora el pedido debe tener varios estados es decir “Enviado”, “Receptado”, “En proceso”, “En camino”, “Finalizado”.






Gestión de cuentas de clientes. - Los clientes de la distribuidora pueden crear una cuenta para poder generar pedidos. Si el cliente no está registrado en la base de datos de la distribuidora no podrá registrarse ni iniciar sesión. El registro se debe realizar con base a la cédula más los datos correspondientes a la cuenta del usuario.


Al igual que en el punto anterior estos cambios son en la interfaz del proyecto por lo cual nos queda de la siguiente manera la parte del registro:















 
En donde se debe llenar los datos para poder completar con el registro, una ves terminando el registro se prosigue a realizar el login del cliente registrado.














 


 

 

CONCLUSIONES:
•	La implementación de los servicios web gracias al postman es una forma de verificar como esta funcionando dicho servicio sin embargo suele dar un poco de problemas ya que en el postman se debe ingresar la url con los path correspondientes y parámetros de ser necesarios ya que si se coloca algo diferente no va a salir el resultado esperado y lo más probable es que de un error.
•
RECOMENDACIONES:
•	Fijarse en colocar los path correctos para cada prueba con el postman ya que esto es necesario para ver el funcionamiento correcto del servicio.


Roberto Pacho							Fernando Deleg

                                                            
