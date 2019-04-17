#include "csapp.h"

void doit(int fd);
void read_requesthdrs(rio_t *rp);
int parse_uri(char *uri,  char *filename,  char *cgiargs);
void serve_static(int fd,  char *filename, int filesize);
void get_filetype(char *filename, char *filetype);
void serve_dynamic(int fd, char *filename, char *cgiargs);
void clienterror(int fd, char *cause, char *errnum, char *shortmsg, char *longmsg);


int main(int argc, char**argv)
{
  int listenfd, connfd, port, clientlen;
  struct sockaddr_in clientaddr;
  
  if(argc != 2){
    fprintf(stderr,"usage: %s <port>\n",argv[0]);
    exit(1);
  }
  
  port = atoi(argv[1]);  //ascii to integer
  
  listenfd = Open_listenfd(port);  //打开一个监听套接字
  
  while(1)
  {
    clientlen = sizeof(clientaddr);
    connfd = Accept(listenfd,(SA*)&clientaddr, &clientlen);  
    doit(connfd);   //处理一个HTTP事务
    Close(connfd);   //关闭已连接套接字
  }
  
}

void doit(int fd)
{
  int is_static;
  struct stat sbuf;
  char buf[MAXLINE],method[MAXLINE],uri[MAXLINE],version[MAXLINE];
  char filename[MAXLINE], cgiargs[MAXLINE];
  rio_t rio;  //rio数据结构
  
  //Read request line and headers  读取请求行和报头
  Rio_readinitb(&rio, fd);  //将文件描述符与rio数据结构的缓冲区绑定起来
  Rio_readlineb(&rio, buf, MAXLINE);
  sscanf(buf, "%s, %s, %s", method, uri, version);
  if(strcasecmp(method,"GET"))
  {
    clienterror(fd,method,"501","Not Implemented","Tiny does not implement this method");
    return;
  }
  read_requesthdrs(&rio);  //读取请求头
  
  //parse URI from GET request
  is_static = parse_uri(uri, filename, cgiargs);
  if(stat(filename, &sbuf)<0){  //获取文件元数据
    clienterror(fd,filename,"404","Not Found","Tiny couldn't find the file");
    return;  
  }
  
  if(is_static)  //serve static content
  {
    if(!(S_ISREG(sbuf.st_mode))||!(S_IRUSR & sbuf.st_mode))
    {
    clienterror(fd,filename,"403","Forbidden","Tiny couldn't read the file");
    return;  
    }
    serve_static(fd,filename,sbuf.st_size);
  }
  else   //serve dynamic content
  {
    if(!(S_ISREG(sbuf.st_mode))||!(S_IXUSR & sbuf.st_mode))
    {
    clienterror(fd,filename,"403","Forbidden","Tiny couldn't run CGI program");
    return;  
    }
    serve_dynamic(fd,filename,cgiargs);  
  }
}


void clienterror(int fd, char *cause, char *errnum, char *shortmsg, char *longmsg)
{
  char buf[MAXLINE], body[MAXBUF];
  
  //Build the HTTP response body
  sprintf(body, "<html><title>Tiny Error</title>");
  sprintf(body, "%s<body bgcolor=""ffffff"">\r\n", body);
  sprintf(body, "%s%s:%s\r\n",body,errnum,shortmsg);
  sprintf(body, "%s<p>%s: %s\r\n", body, longmsg, cause);
  sprintf(body, "%s<hr><em>The Tiny Web server</em>\r\n",body);
  
  
  //Print the HTTP reponse
  sprintf(buf, "HTTP/1.0 %s %s\r\n", errnum, shortmsg);
  Rio_writen(fd,buf,strlen(buf));
  sprintf(buf, "Conten-type: text/html\r\n");
  Rio_writen(fd,buf,strlen(buf));
  sprintf(buf,"Content-length: %d\r\n\r\n",(int)strlen(body);
  Rio_writen(fd,buf,strlen(buf));
  Rio_writen(fd,body,strlen(buf));
}


void read_requesthdrs(rio_t *rp)
{
  char buf[MAXLINE];
  
  Rio_readlineb(rp,buf,MAXLINE);
  while(strcmp(buf,"\r\n")){
    Rio_readlineb(rp,buf,MAXLINE);
    printf("%s",buf);
  }
  return;
}

int parse_uri(char *uri, char *filename, char *cgiargs)
{
  char *ptr;
  
  if(!strstr(uri,"cgi-bin"))  //static content
  {
    strcpy(cgiargs,"");
    strcpy(filename,".");
    strcat(filename,uri);
    if(uri[strlen(uri)-1] =='/')
      strcat(filename, "home.html");
    return 1;
    
  }
  else{  //dynamic content
    ptr = index(uri, '?');
    if(ptr){
      strcpy(cgiargs, ptr+1);
      *ptr = '\0';
    }
    else
      strcpy(cgiargs,"");
    strcpy(filename,".");
    strcpy(filename,uri);
    return 0;
  }
}





