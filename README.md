# Entrega-6.2
#include <string.h> 
#include <unistd.h> 
#include <stdlib.h>
#include <sys/types.h>  
#include <sys/socket.h> 
#include <netinet/in.h> 
#include <stdio.h>  
#include <pthread.h>  
int contador;   
//Estructura necesaria para acceso excluyente  
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;   
int i; 
int sockets[100];    
void *AtenderCliente (void *socket) 
{   int sock_conn;   
    int *s;   
    s= (int *) socket;  
    sock_conn= *s;      
    int socket_conn = * (int *) socket;      
    char peticion[512];   
    char respuesta[512];  
    int ret;         
    int terminar =0;   
    // Entramos en un bucle para atender todas las peticiones de este cliente hasta que se desconecte   
    while (terminar ==0)   
    {    
    // Ahora recibimos la peticion    
       ret=read(sock_conn,peticion, sizeof(peticion));   
       printf ("Recibido\n");       
       // Tenemos que añadirle la marca de fin de string  para que no escriba lo que hay despues en el buffer   
       peticion[ret]='\0';            
       printf ("Peticion: %s\n",peticion);        
       // vamos a ver que quieren   
       char *p = strtok( peticion, "/");    
       int codigo =  atoi (p);    
       int numForm;    /
       /Ya tenemos el c?digo de la petici?n   
       char nombre[20];        
       if (codigo !=0)  
       {
           p = strtok( NULL, "/");     
           numForm =  atoi (p);     
           p = strtok( NULL, "/");     
           strcpy (nombre, p);     
      // Ya tenemos el nombre     
           printf ("Codigo: %d, Nombre: %s\n", codigo, nombre);    
           
        }        
        if (codigo ==0) //petici?n de desconexion     
            terminar=1;    
        else if (codigo ==1) //piden la longitd del nombre     
             sprintf (respuesta,"1/%d/%d",numForm,strlen (nombre));    
        else if (codigo ==2)     // quieren saber si el nombre es bonito     
             if((nombre[0]=='M') || (nombre[0]=='S'))     
             sprintf (respuesta,"2/%d/SI", numForm);     
             else      
             sprintf (respuesta,"2/%d/NO", numForm);    
             else //quiere saber si es alto     
             {     
                p = strtok( NULL, "/");      
                float altura =  atof (p);     
                if (altura > 1.70)       
                sprintf (respuesta, "3/%d/%s: eres alto",numForm,nombre);      
              }
            else      
               sprintf (respuesta, "3/%d/%s: eres bajo",numForm, nombre);     
            if (codigo !=0)     
            {            
               printf ("Respuesta: %s\n", respuesta);      // Enviamos respuesta      
               write (sock_conn,respuesta, strlen(respuesta));     
            }     
            if ((codigo ==1)||(codigo==2)|| (codigo==3))     
            {      
            pthread_mutex_lock( &mutex ); //No me interrumpas ahora      
            contador = contador +1;      
            pthread_mutex_unlock( &mutex); //ya puedes interrumpirme      
            // notificar a todos los clientes conectados     
            char notificacion[20];      
            sprintf (notificacion, "4/%d",contador);     
            int j;      
            for (j=0; j< i; j++)       
            write (sockets[j],notificacion, strlen(notificacion));           
            }       
        }  
        // Se acabo el servicio para este cliente  
        close(sock_conn);     
  }  

         
  
