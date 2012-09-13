Progra-1-Lenguajes

//Desarrollo de la 1 Tarea Programada

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/utsname.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <unistd.h>
#include <fcntl.h>
#include "progra.h"

typedef struct sockaddr *sad;
 void error(char *s) {
	exit((perror(s), -1));
	}
/*El main es la función inicial, aqui se inicia el fork que separara la parte del servidor y la del cliente. Si el resultado de la función es mayor
 o igual a cero significa que la conexión existe y depende si es un cero o no este trbajará cómo cliento o servidor*/
int main () {
	pid_t hijo;
	hijo = fork();
	if (hijo >= 0) {
		if (hijo == 0) {
			CL("172.26.97.77", 5000);
			}
		else {
			SL(5000);
			}
		}
	else {
		perror("fork");
		exit(0);
		}
	return 0;
	}

/*Función cliente, recibe una dirección IP y el número de puertos que acepta el servidor. Se define el puerto de trabajo, un int para el socket, 
 * la estructura de trabajo para el socket, y un arreglo para insertar el mensaje en él*/

int CL(char *ip, int puerto) {
	#define PORT1 puerto
	int sock;
    	struct sockaddr_in sin;
    	char linea[1024];
    	int largo =128;    

    	/*Se inicia la coneccion del cliente para que reconozca al servidor, si no es de esa manera envía la palabra socket a la función error 
     	* para informar al respecto*/

    	if ((sock = socket(AF_INET, SOCK_STREAM, 0))<0)
        	error("socket");
        sin.sin_family = AF_INET;
        sin.sin_port = htons(PORT1);
        inet_aton(ip, &sin.sin_addr);
        
        /* Inicia la llamada o conexión entre el cliente y el servidor, de no cumplirse se envía connect a la función error*/
        
        if (connect(sock, (sad) &sin, sizeof(sin))<0)
               error("connect");
        
        /*Inicio de la comunicación */
        
        int sesion = 0;
        while (sesion != 1) {
        	pid_t hijopid;
		
		/* Inicio de la Bifurcacion con este sección se facilita la función de enviar y recibir mensajes de manera independiente, en otras palabras no se debe
		* esperar respuesta por el servidor ni viceversa, haciendo al programa cliente y servidor*/
			 
		hijopid = fork();

		/*parte hijo del fork*/

		if (hijopid >= 0) {	
			if (hijopid == 0) {
				char mensaje[1024];
				printf("\e[34;01m-");
				gets (mensaje);
				if (write (sock,&mensaje,largo) <0 )
					error("write");
				if(strcmp(mensaje,"Adios")==0) {
					return 0;
					close(sock);
					exit(0);
					break;
					system("pause");
					return EXIT_SUCCESS;
					}	
				}

			/*Parte padre del fork*/

			else {
				if ((largo = read(sock, linea, sizeof(linea))) < 0)
					error ("read");
				if(strcmp(linea, "Adios")==0) {
					printf("Hasta la proxima!");
					close(0);
					exit(0);
					break;
					system("pause");
					return EXIT_SUCCESS;
					}
				linea[largo]=0;
				printf("\e[35;01m servidor: %s \n", linea);	
				}
			}

		/*Retorna error si el fork falla enviando la palabra fork a la función perror*/

		else {
			perror("fork");
			exit(0);
			}
		}
      return 0;
      }

/*Función SL recibe el puerto*/

int SL(int puerto) {
	#define PORT2 puerto
	int sock, sock1;
    	struct sockaddr_in sin, sin1;
    	char linea[1024];
    	socklen_t conecta;
    	int largo = 128;
    	if ((sock = socket (AF_INET, SOCK_STREAM, 0)) <0)
		error("socket");
        
        /*Se inicia el servidor a la espera de la conexión de algún cliente */
        
        memset(&sin, 0, sizeof sin);
        sin.sin_family = AF_INET;
        sin.sin_port = htons(PORT2);
        sin.sin_addr.s_addr = INADDR_ANY;
 
        /*Se envía la palabra bind a la función error si la busqueda no es acertada y así informarnos al respecto*/

        if(bind(sock, (sad) &sin, sizeof sin)<0)
		error ("bind");
 
        /* Se define el número maximo de clientes con los que se realizará la conexión*/

        if (listen(sock, 5) < 0)
                error ("listen");

        /* Se mantendrá al servidor a la espera de un cliente, de no conectarse ninguno este enviará la palabra accept a la función error*/

        conecta = sizeof(sin1);
        if((sock1 = accept(sock, (sad)&sin1, &conecta)) < 0)
		error("accept");

	while (linea != 0) {
		pid_t hijopid;

		/* Inicio de la Bifurcacion con este sección se facilita la función de enviar y recibir mensajes de manera independiente, en otras palabras no se debe
		 * esperar respuesta por el servidor ni viceversa, haciendo al programa cliente y servidor*/		

		hijopid = fork();
		if (hijopid >= 0) {

			/*Parte hijo del fork*/

			if (hijopid == 0) {
				printf("\e[34;01m-");
				gets (linea);
				if (write(sock1, linea, largo) < 0)
					error("write");
				if(strcmp(linea,"Adios")==0) {
					close(sock1);
					close(sock);
					exit(0);
					break;
					system("pause");
					return EXIT_SUCCESS;
					}
				}
			/*Parte padre del fork*/

			else {
				if ((largo = read(sock1, linea, sizeof(linea)))<0)
                        		error("read");
                    		if(strcmp(linea, "Adios")==0) {
					printf("%s",linea);
					close(sock1);
					close(sock);
					exit(0);
					break;
					system("pause");
					return EXIT_SUCCESS;
					}
				linea[largo] = 0;
                    		printf("\e[35;01m cliente: %s \n", linea);
				linea[0]++;
				}
			}
		/*Envía la palabra fork a la función perror en caso de que la función falle*/

		else {
			perror("fork");
			exit(0);
			}
		}
      return 0;
      }

/*Apartir de aqui inicia el codigo del archivo progra.h*/

#ifndef MES_H
#define MES_H
int CL(char *ip,int puerto);
int SL(int puerto2);
#endif