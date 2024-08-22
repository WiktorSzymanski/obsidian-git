---
up: "[[Gniazda Sieciowe]]"
class: PS
---
# Podstawowe Gniazda Sieciowe
---
pozwalają na komunikację _strumieniową_ (_TCP/IP_) i _datagramową_ (_UDP/IP_). W przeciwieństwie do gniazd niższych poziomów, do ich użycia nie są wymagane uprawnienia _root_-a.

![[Pasted image 20240309155711.png]]
Można je utworzyć przy pomocy funkcji systemowej [[socket(2)]].

#### Przykłady
> [!Box]- Serwer TCP
> ``` C
int sfd, cfd;
socklen_t sl;
struct sockaddr_in saddr, caddr;
memset(&saddr, 0, sizeof(saddr));
saddr.sin_family = AF_INET;
saddr.sin_addr.s_addr = INADDR_ANY;
saddr.sin_port = htons(1234);
sfd = socket(PF_INET, SOCK_STREAM, 0);
bind(sfd, (struct sockaddr*) &saddr, sizeof(saddr));
listen(sfd, 5);
while(1) {
>	sl = sizeof(caddr); cfd = accept(sfd, (struct sockaddr*) &caddr, &sl);
>	write(cfd, "Hello World!\n", 13);
>	close(cfd);
>}
> ```

> [!Box]- Klient TCP
>``` C
int sfd;
char buf[128];
struct sockaddr_in saddr;
struct hostent* addrent;
addrent = gethostbyname(argv[1]);
memset(&saddr, 0, sizeof(saddr));
saddr.sin_family = AF_INET;
saddr.sin_port = htons(atoi(argv[2]));
memcpy(&saddr.sin_addr.s_addr, addrent->h_addr,
addrent->h_length);
sfd = socket(PF_INET, SOCK_STREAM, 0);
connect(sfd, (struct sockaddr*) &saddr, sizeof(saddr));
read(sfd, buf, sizeof(buf));
close(sfd);
>```

> [!Box]- Serwer UDP
>``` C
int sfd; socklen_t sl; char buf[128];
struct sockaddr_in saddr, caddr;
memset(&saddr, 0, sizeof(saddr));
saddr.sin_family = AF_INET;
saddr.sin_addr.s_addr = INADDR_ANY;
saddr.sin_port = htons(1234);
sfd = socket(PF_INET, SOCK_DGRAM, 0);
bind(sfd, (struct sockaddr*) &saddr, sizeof(saddr));
while(1) {
>	memset(&caddr, 0, sizeof(caddr));
>	memset(&buf, 0, sizeof(buf));
>	sl = sizeof(caddr);
>	recvfrom(sfd, buf, 128, 0, (struct sockaddr*) &caddr, &sl);
>	sendto(sfd, buf, strlen(buf)+1, 0,
>	(struct sockaddr*) &caddr, sl);
}
>```

> [!Box]- Klient UDP
>``` C
int sfd;
socklen_t sl;
char buf[128];
struct sockaddr_in saddr, caddr;
struct hostent* addrent;
addrent = gethostbyname(argv[1]);
memset(&caddr, 0, sizeof(caddr));
caddr.sin_family = AF_INET;
caddr.sin_addr.s_addr = INADDR_ANY;
caddr.sin_port = 0;
memset(&saddr, 0, sizeof(saddr));
saddr.sin_family = AF_INET;
memcpy(&saddr.sin_addr.s_addr, addrent->h_addr,
addrent->h_length);
saddr.sin_port = htons(atoi(argv[2]));
sfd = socket(PF_INET, SOCK_DGRAM, 0);
bind(sfd, (struct sockaddr*) &caddr, sizeof(caddr));
strcpy(buf, "Hello server!\n");
sendto(sfd, buf, strlen(buf)+1, 0, (struct sockaddr*) &saddr,
sizeof(saddr));
sl = sizeof(saddr);
recvfrom(sfd, buf, 128, 0, (struct sockaddr*) &saddr, &sl);
close(sfd);
>```

>Funkcja systemowa `gethostbyname(2)` we współczesnych implementacjach powinna być
zastąpiona funkcją systemową `getaddrinfo(3)`.
Funkcja systemowa `gethostbyname(2)` występuje w przykładach tylko na potrzeby
zwięzłości implementacji.