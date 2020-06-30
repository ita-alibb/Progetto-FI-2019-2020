# Progetto-FI-2019-2020
Progetto Andrea Aliberti
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define Max 26
#define puliscibuffer while  ((getchar())!='\n')

//DEFINISCO LE STRUTTURE DATI
//Data
typedef struct {
	int gg;
	int mm;
	int aaaa;
} Data;

//Genere
typedef struct {
	char Genere1[Max];
	char Genere2[Max];
	char Genere3[Max];
} Genere;
//Libro
typedef struct{
	char Titolo[Max*2];
	char Autore[Max+10];
	Genere G;
	int Nlibri;
	int Codice;
} Libro;

//Libro in prestito
typedef struct{
	char Username[Max];
	int Codice;
	Data Data_reso;
} Libro_prestato;

//DEFINISCO LE LISTE
//Lista dei libri
struct ListaLibri{
        Libro L;
        struct ListaLibri* next;
};
typedef struct ListaLibri ListaLibri;

//Libri in prestito Amministratore/Utente e Storico
struct ListaLP{
        Libro_prestato LibroP;
        struct ListaLP* next;
};
typedef struct ListaLP ListaLP;

//Definisco la struct Utente che contiene anche la ListaLP
//Utente
typedef struct {
        char Username[Max];
        char Password[Max];
        ListaLP* L_prestito;
} Utente;

//Lista per gli Utenti
struct ListaU {
	Utente user;
	struct ListaU* next;
} ;
typedef struct ListaU ListaU;

//FUNZIONI UTILI AL PROGRAMMA:
//Funzione per le potenze
int pot(int a,int b){
        int p=1;
        for (b;b>0;b--){
        p=p*a;
        }
        return p;
}

//Funzione per generare un codice Univoco
int generacodice(ListaLibri* head){
	int i;
	int Code;
	int flag=0;

	//genero il codice di 5 cifre
        srand(time(NULL));
       	Code=(rand()%(pot(10,5)));
	//Controllo che le cifre siano 5
	if (Code<10000){
		Code+=10000;
	}

	//controlla che non sia gia' in uso, nel caso fosse gia' in uso ne genera uno nuovo e ricontrolla
	while (flag==0){
		while (head!=NULL && head->L.Codice!=Code){
			head=head->next;
		}
		if (head==NULL){
			flag=1;
		} else if (head->L.Codice==Code){
			Code++;
		}
	}

	return Code;
}
//Funzione per aggiungere un mese alla data
void mesepiu(Data* D){
        D->mm+=1;
        if ((D->mm)==13){
                D->mm=1;
                D->aaaa+=1;
        }
}

//Funzione per aggiungere un giorno alla data
void giornopiu(Data* D){
        D->gg+=1;
        if ((D->gg)==31){
		D->gg=1;
		mesepiu(D);
	}

}

//Funzione che riconosce se la stringa inserita e' un codice o un titolo
        //Se la stringa e' più lunga di 5 (lunghezza dei codici) si esclude la possibilita' che sia un codice
int riconosci(char* oggetto,int lun){
        int i;
        int flag=1;

        if (strlen(oggetto)!=lun){
                return 2;
        }
        for (i=0;i<lun;i++){
                if ((int)oggetto[i]<48 || (int)oggetto[i]>57){
                        flag=2;
                }
        }
        return flag;
}

//Funzione che converte una stringa di 4 numeri in un numero
int converti(char* oggetto){
        int numero=0;
        int i;
	int N=0;
	N=strlen(oggetto);
        for (i=0;i<N;i++){
                numero=numero+((int)oggetto[i]-48)*pot(10,(N-1)-i);
        }
        return numero;
}

//Funzione per comparare due date: restituisce la differenza in giorni tra la prima e la seconda. 0 se uguali >0 se piu' lontana la prima <0 se e' piu' lontana la seconda
int ComparaDate(Data Data1,Data Data2){
        int Diff=0;


        Diff=(Data1.aaaa-Data2.aaaa)*365+(Data1.mm-Data2.mm)*30+(Data1.gg-Data2.gg);

        return Diff;
}

//Funzione per inizializzare correttamente le stringhe da fgets
void leva(char* oggetto){
	if (oggetto[strlen(oggetto)-1]!='\n'){
		puliscibuffer;
	}

	oggetto[strlen(oggetto)-1]='\0';
}

//Funzione di controllo per le stringhe,flag=0 stringa senza spazi, flag=1 stringa con spazi, in entrambe il controllo sulla lunghezza
void string_check(char* saved, int flag){
	int i=0,flag2=0,flag3=0;

	//se l'ultimo carattere della fgets non e' '\n' allora la stringa inserita e' troppo lunga
	if (saved[strlen(saved)-1]!='\n'){
		//svuoto il buffer
		puliscibuffer;
		flag3=1;
	} else {
		//sostituisco il carattere '\n' con '\0' per terminare correttamente la stringa
		saved[strlen(saved)-1]='\0';
	}

	//controllo che non ci siano spazi se la flag e' 0
        if (flag==0){
                for (i=0;i<strlen(saved);i++){
                        if (saved[i]==' '){
                                saved[i]='\0';
                                flag2=1;
                                break;
                        }
                }
        }

	if (flag3==1 && flag2==1){
			printf("\tE' stata inserita una stringa troppo lunga e con degli spazi, la stringa salvata e': %s !\n",saved);
	}else if (flag3==1 && flag2==0){
			printf("\tE' stata inserita una stringa troppo lunga, la stringa salvata e': %s !\n",saved);
	} else if (flag3==0 && flag2==1){
			printf("\tE' stata inserita una stringa con degli spazi, la stringa salvata e': %s !\n",saved);
	}
}
//Funzione per cambiare la disponibilita' di un libro
void cambiadisp(ListaLibri** head,Libro L,int op){
	ListaLibri* temp=*head;

	while(temp!=NULL && strcmp(temp->L.Titolo,L.Titolo)!=0){
		temp=temp->next;
	}
	if (temp!=NULL){
		if (op==0){
			temp->L.Nlibri--;
		} else if (op==1){
			temp->L.Nlibri++;
		}
	}
}

//FUNZIONI PER INIZIALIZZARE LE STRUTTURE DATI
void inizializzaD(Data *D){
	char oggetto[10];
	int flag;

	do{
		flag=0;

		printf("Giorno (gg):");
		leva(fgets(oggetto,10,stdin));
		if (riconosci(oggetto,2)!=1 && riconosci(oggetto,1)!=1){
			flag=1;
			printf("Inserire il formato corretto!(gg)\n");
		} else {
			D->gg=converti(oggetto);
			if (D->gg<1 || D->gg>30){
                                printf("I mesi vanno da 1 a 30\n");
                                flag=1;
                        }
		}
	} while (flag!=0);

	do{
		flag=0;

		printf("Mese (mm):");
	       	leva(fgets(oggetto,10,stdin));
	        if (riconosci(oggetto,2)!=1 && riconosci(oggetto,1)!=1){
	                flag=1;
	                printf("Inserire il formato corretto!(mm)\n");
	        } else {
		        D->mm=converti(oggetto);
			if (D->mm<1 || D->mm>12){
				printf("I mesi vanno da 1 a 12\n");
				flag=1;
			}
		}
	} while (flag!=0);

	do{
		flag=0;

		printf("Anno (aaaa):");
	        leva(fgets(oggetto,10,stdin));
	        if (riconosci(oggetto,4)!=1){
	                flag=1;
	                printf("Inserire il formato corretto!(aaaa)\n");
	        } else {
		        D->aaaa=converti(oggetto);
			if (D->aaaa<1){
                                printf("L'anno deve essere positivo\n");
                                flag=1;
                        }
		}
	} while (flag!=0);

}

//funzione per inizializzare Utente
void inizializzaU(Utente* U){

	printf("\nUsername(non deve contenere spazi):\n");
	fgets(U->Username,Max,stdin);
	string_check(U->Username,0);

	printf("Password(non deve contenere spazi):\n");
	fgets(U->Password,Max,stdin);
	string_check(U->Password,0);

	U->L_prestito=NULL;
}

//funzione per inizializzare il genere
void inizializzaG(Genere* G) {

	//Primo Genere
	printf("Primo Genere:");
	fgets(G->Genere1,Max,stdin);
	string_check(G->Genere1,0);

	if (strcmp(G->Genere1,"Vuoto")==0){
		printf("Il libro deve avere almeno un genere!");
		while (strcmp(G->Genere1,"Vuoto")==0){
			printf("Primo Genere: ");
			fgets(G->Genere1,Max,stdin);
			string_check(G->Genere1,0);
		}
	}

	//Secondo Genere
	printf("Secondo Genere: ");
	fgets(G->Genere2,Max,stdin);
	string_check(G->Genere2,0);

	if (strcmp("Vuoto",G->Genere2)==0){
		strcpy(G->Genere2,"-");
		strcpy(G->Genere3,"-");
		return ;
	}
	//Terzo Genere
	printf("Terzo Genere: ");
	fgets(G->Genere3,Max,stdin);
	string_check(G->Genere3,0);

	if (strcmp(G->Genere3,"Vuoto")==0){
		strcpy(G->Genere3,"-");
		return ;
	}
	return ;
}

//funzione per inizializzare Libro
void inizializzaL(Libro* L,ListaLibri* LisL){
	int n=0;

	printf("Titolo: ");
	fgets(L->Titolo,Max*2,stdin);
	string_check(L->Titolo,1);

	printf("Autore: ");
	fgets(L->Autore,Max+10,stdin);
	string_check(L->Titolo,1);

	inizializzaG(&(L->G));

	printf("Disponibilita':");
	scanf("%d",&n);
	if (n<0){
		n=n*(-1);
		printf("La disponibilita' e' stata resa positiva poiche' e' stata inserita una quantita' negativa!");
	}
	L->Nlibri=n;
	L->Codice=generacodice(LisL);
}

//funzione per inizializzare Libro prestato
void inizializzaLP(Libro_prestato* Lp,char* UtLog,int Codice,Data D){

	strcpy(Lp->Username,UtLog);

	Lp->Codice=Codice;
	mesepiu(&D);
	Lp->Data_reso=D;

}

//FUNZIONI CHE CREANO LE LISTE
//Lista dei libri
ListaLibri* CreaListaLibri(Libro L){
	ListaLibri* head;

	head=(ListaLibri*)malloc(sizeof(ListaLibri));

	head->L=L;
	head->next=NULL;

	return head;
}

//Lista dei libri in prestito
ListaLP* CreaListaLP(Libro_prestato Lp){
	ListaLP* head;

	head=(ListaLP*)malloc(sizeof(ListaLP));

	head->LibroP=Lp;
	head->next=NULL;

	return head;
}
//Lista degli Utenti
ListaU* CreaListaU(Utente U){
	ListaU* head;

	head=(ListaU*)malloc(sizeof(ListaU));

	head->user=U;
	head->next=NULL;

	return head;
}

//FUNZIONI PER L'INSERIMENTO NELLE LISTE
//Inserimento Ordinato ListaU (chiave= Username) (0 successo,1 errore)
int inserisciListaU(ListaU** head,Utente U){

	//Controllo che non sia possibile prendere come Username l'ID dell'Admin
	if(strcmp(U.Username,"ADMINID")==0){
		printf("L'Username e' gia' in uso, cambiare Username");
		return 1;
	}

	ListaU* n;
	n=(ListaU*)malloc(sizeof(ListaU));

	ListaU* p;
	p=*head;

	ListaU* t;
	t=*head;

	n->user=U;
	//'='non e' stato inserito per controllare che l'username non sia gia' stato preso
	while(t!=NULL && strcmp(t->user.Username,U.Username)<0 ){
		p=t;
		t=t->next;
	}
	if (t==*head){
		n->next=*head;
		*head=n;
	} else if (t==NULL || strcmp(t->user.Username,U.Username)>0){
		n->next=t;
		p->next=n;
	} else if (strcmp(t->user.Username,U.Username)==0){
		printf("L'Username e' gia' in uso. Cambiare username.");
		return 1;
	}
	return 0;
}


//Inserimento ordinato per la ListaLibri (chiave= Titolo) (0 libro non presente,1 libro gia' presente)
int inserisciListaLibri(ListaLibri** head,Libro Lib){

	ListaLibri* n;
	n=(ListaLibri*)malloc(sizeof(ListaLibri));

	ListaLibri* t;
	t=*head;

	ListaLibri* p;
	p=*head;

	n->L=Lib;
	//'='non e' presente poiche' nel caso in cui il libro sia gia' presente si deve solo aggiungere la disponibilita'
	while (t!=NULL && strcmp(t->L.Titolo,Lib.Titolo)<0){
	p=t;
	t=t->next;
	}
	if (t==*head){
		n->next=*head;
		*head=n;
	} else if (t==NULL || strcmp(t->L.Titolo,Lib.Titolo)>0){
		n->next=t;
		p->next=n;
	} else if (strcmp(t->L.Titolo,Lib.Titolo)==0){
		t->L.Nlibri+=Lib.Nlibri;
		return 1;
	}
	return 0;
}

//Inserimento Ordinato ListaLP (chiave Username)
void inserisciLP(ListaLP** head,Libro_prestato Lp){
	ListaLP* n;
	n=(ListaLP*)malloc(sizeof(ListaLP));

	ListaLP* t;
	t=*head;

	ListaLP* p;
	p=*head;

	n->LibroP=Lp;
	//Cerco il primo Username uguale all'Username di Lp
	while (t!=NULL && strcmp(t->LibroP.Username,Lp.Username)<0){
	p=t;
	t=t->next;
	}

	if (t==*head){//Caso particolare in cui bisogna inserirlo in testa
		n->next=*head;
		*head=n;
	} else {
		p->next=n;
		n->next=t;
	}
}

//inizializza LibriInPrestito (admin)
void inizializzaListaLPadmin (ListaLP** admin,ListaU* Utenti){

        ListaLP* headU;

        while (Utenti!=NULL){//Per ogni Utente della lista utente
                headU=Utenti->user.L_prestito;

                while (headU!=NULL){//Scorri la lista di libri prestati fino alla fine, dove fine==NULL e inizio==ultimo nodo, inzio->next==fin$
                        inserisciLP(admin,headU->LibroP);
                        headU=headU->next;
                }

                Utenti=Utenti->next;//Scorri all'utente successivo
        }

}


//Funzioni per la ricerca
//Libri
	//Per Titolo
ListaLibri* cercaTitolo(ListaLibri* head,char* titolo){
	ListaLibri* p;
	p=head;

	while(head!=NULL && strcmp(head->L.Titolo,titolo)<0){
		p=head;
		head=head->next;
	}

	return p;

}
	//Per Codice
ListaLibri* cercaCodice(ListaLibri* head,int Code){
        ListaLibri* p;
	p=head;

        while(head!=NULL && head->L.Codice!=Code){
                p=head;
		head=head->next;
        }

        return p;
}

	//Filtra titolo (ricerca del titolo anche per parte di esso)
void FiltraT (ListaLibri* head,char* Titolo){
	int N=0,count=0;
	N=strlen(Titolo);

	while (head!=NULL){
		if (strncmp(Titolo,head->L.Titolo,N)==0){
			printf("\nTitolo:%s\nAutore:%s\nGeneri:\n%s %s %s\nDisponibilita':%d\nCodice:%d\n",head->L.Titolo,head->L.Autore,head->L.G.Genere1,head->L.G.Genere2,head->L.G.Genere3,head->L.Nlibri,head->L.Codice);
			count++;
		}
		head=head->next;
	}
	if (count==0){
		printf("\n\tNessun Libro soddisfa i criteri di ricerca.\n\n");
	}
}

	//Filtro lista per Genere
void FiltraG(ListaLibri* head,char* Gen){
	int count=0;
	while(head!=NULL){
		if (strcmp(head->L.G.Genere1,Gen)==0 || strcmp(head->L.G.Genere2,Gen)==0 || strcmp(head->L.G.Genere3,Gen)==0) {
			printf("\nTitolo:%s\nAutore:%s\nGeneri:\n%s %s %s\nDisponibilita':%d\nCodice:%d\n",head->L.Titolo,head->L.Autore,head->L.G.Genere1,head->L.G.Genere2,head->L.G.Genere3,head->L.Nlibri,head->L.Codice);
			count++;
		}
		head=head->next;
	}
	if (count==0){
		printf("\n\tNessun Libro soddisfa i criteri di ricerca.\n\n");
	}
}

	//Filtro lista per Autore
void FiltraA(ListaLibri* head,char* Aut){
	int count=0;
	while(head!=NULL){
		if (strcmp(head->L.Autore,Aut)==0){
			printf("\nTitolo:%s\nAutore:%s\nGeneri:\n%s %s %s\nDisponibilita':%d\nCodice:%d\n",head->L.Titolo,head->L.Autore,head->L.G.Genere1,head->L.G.Genere2,head->L.G.Genere3,head->L.Nlibri,head->L.Codice);
			count++;
		}
		head=head->next;
	}
	if (count==0){
		printf("\n\tNessun Libro soddisfa i criteri di ricerca.\n\n");
	}
}

//Ricerca Libro_prestato (restituisce 0 se il libro non e' presente e 1 se il libro e' presente)
int cercaLP(ListaLP* head,int Codice){
	while (head!=NULL && head->LibroP.Codice!=Codice){
		head=head->next;
	}
	if (head==NULL){
		return 0;
	} else {
		return 1;
	}
}

//Utente return 1 se esiste; return 2 se non esiste; return 3 se la password e' sbagliata; return 0 se sono state inserite le credenziali dell'Admin;
int LogU(ListaU* head,Utente* Ut){
        ListaU* p;

	if ((strcmp(Ut->Username,"ADMINID")==0) && (strcmp(Ut->Password,"ADMINPASS")==0)){
		return 0;
	}else{

	        while(head!=NULL && strcmp(head->user.Username,Ut->Username)!=0){
	                p=head;
	                head=head->next;
	        }
		if (head==NULL){
			return 2;
		} else if (strcmp(head->user.Password,Ut->Password)==0){
			Ut->L_prestito=head->user.L_prestito;//associo la lista di libri in prestito
			return 1;
		} else {
			return 3;
		}
	}
}


//Funzioni di rimozione
//Elimina Utente
int EliminaU(ListaU** head,Utente Utent){
	ListaU* p;
	ListaU* t=*head;
	char c='0';

	while(t!=NULL && strcmp(t->user.Username,Utent.Username)<0){
		p=t;
		t=t->next;
	}
	if (t==NULL || strcmp(t->user.Username,Utent.Username)!=0 ){
		printf("\nL'Utente non e' presente\n");
		return 1;
	} else if (strcmp(t->user.Password,Utent.Password)==0 || strcmp(Utent.Password,"ADMINPASS")==0){
		if (t->user.L_prestito!=NULL){
			printf("L'utente in questione ha dei libri da riconsegnare!\nForzare la cancellazione?(i libri in prestito verranno persi.)\nY/N:");
			c=getchar();
		}
		if (c=='0' || c=='Y' || c=='y'){
			if (t==*head){
				*head=t->next;
				free(t);
				printf("L'Utente e' stato eliminato correttamente.\n");
			} else {
				p->next=t->next;
				free(t);
				printf("L'Utente e' stato eliminato correttamente.\n");
			}
		}
	}
	return 0;
}

//Elimina Libro (lcercato e' il nodo precedente al libro che deve essere eliminato, e' cio' che ritorna la funzione per cercare i libri);
void EliminaL(ListaLibri** lcercato,ListaLibri** head,char* Titolo,ListaLP* LibriInPrestito){
	char password[Max];
	int codice;

	ListaLibri* p=*lcercato;
	ListaLibri* t=(p->next);

	printf("Password Admin: ");
	fgets(password,Max,stdin);
	string_check(password,0);


	if (strcmp(password,"ADMINPASS")==0){
		if (p==*head && strcmp((*lcercato)->L.Titolo,Titolo)==0){//Controllo per vedere se l'elemento da eliminare e' la testa della lista
			codice=(*lcercato)->L.Codice;
			if (cercaLP(LibriInPrestito,codice)==0){//Controllo che il libro da eliminare non sia attualmente in prestito
				*head=t;
				free(p);
				printf("Il Libro e' stato eliminato correttamente.\n");
			} else {
				printf("Impossibile eliminare il libro perche' e' attualmente in prestito.\n");
			}
		} else {
			codice=(*lcercato)->next->L.Codice;
			if (cercaLP(LibriInPrestito,codice)==0){//Controllo che il libro da eliminare non sia attualmente in prestito
		                p->next=t->next;
        		        free(t);
				printf("Il Libro e' stato eliminato correttamente.\n");
			}else {
				printf("Impossibile eliminare il libro perche' e' attualmente in prestito.\n");
			}
		}
        } else {
		printf("Password Errata.\n");
	}
}

//Elimina Libro Prestato
void EliminaLP(ListaLP** head, char* Username,int Codice,Data Odierna){
	int diff;
	ListaLP* p;
	ListaLP* t=*head;

	//Vedo se esiste il libro e se e' associato all'utente corretto
	while(t!=NULL){
		if( t->LibroP.Codice==Codice && strcmp(t->LibroP.Username,Username)==0) {
			break;
		}
		p=t;
		t=t->next;
	}
	//Se esiste lo elimino
	if (t==NULL){
		printf("Il Libro non esiste.\n");
	} else {
		if (t==*head){
			*head=t->next;
			free(t);
		} else {
			p->next=t->next;
			free(t);
		}

		diff=(ComparaDate(t->LibroP.Data_reso,Odierna));

                printf("Il Libro e' stato riconsegnato.\n");

		if (diff<0){
			printf("La multa da pagare a causa del ritardo e' di %d $\n",-diff);
		}
        }
}

//Stampa delle Liste
//Stampa ListaLibri
void stampaL(Libro L){
	printf("\nTitolo:%s\nAutore:%s\nGeneri:\n%s %s %s\nDisponibilita':%d\nCodice:%d\n",L.Titolo,L.Autore,L.G.Genere1,L.G.Genere2,L.G.Genere3,L.Nlibri,L.Codice);
}

void stampaListaLibri(ListaLibri* head){
	while(head!=NULL){
		stampaL(head->L);
		head=head->next;
	}
}

//Stampa ListaLP
void stampaLP(Libro_prestato Lp){
	printf("\nUsername:%s\nCodice del Libro preso in prestito:%d\nDa restituire entro: %d/%d/%d\n",Lp.Username,Lp.Codice,Lp.Data_reso.gg,Lp.Data_reso.mm,Lp.Data_reso.aaaa);
}

void stampaListaLP(ListaLP* head){
	while(head!=NULL){
		stampaLP(head->LibroP);
		head=head->next;
	}
}

//FILE
//Creazione dei File (Crea un file col nome inserito)
void creafile(char* nome){
	FILE*f;
	f=fopen(nome,"a");
	fclose(f);
}

//Lettura Lista Utenti
void letturaListaU(ListaU** head){
	FILE* f;
	Utente U;
	char controllo[Max];
	Libro_prestato Lp;

	f=fopen("ListaU.txt","r");

	U.L_prestito=NULL;

	if (fscanf(f,"\n%s\n%s\n",U.Username,U.Password)!=EOF){//leggo il primo username e la prima password
		while (fscanf(f,"\n%s",controllo)!=EOF){//leggo la riga successiva fino a quando non finisce il file
			if (strcmp(controllo,"Libro_prestato:")==0){//Se la riga successiva e' "Libro_prestato:" significa che l'utente ha dei libri n prestito, lo inizializza e lo inserisce nella lista dell'utente 
				strcpy(Lp.Username,U.Username);
				fscanf(f,"\n%d\n%d/%d/%d\n",&(Lp.Codice),&(Lp.Data_reso.gg),&(Lp.Data_reso.mm),&(Lp.Data_reso.aaaa));
        	        	inserisciLP(&(U.L_prestito),Lp);

			} else {//Se invece non e' "Libro_prestato:" significa che o sono finiti i libri in prestito o non ne ha mai avuti
				inserisciListaU(head,U);//inserisce la struttura utente nella lista
				strcpy(U.Username,controllo);//controllo contiene un Username, ma dell'utente successivo
				fscanf(f,"\n%s\n",U.Password);//prende la seconda stringa che sara' quindi la password
				U.L_prestito=NULL;//porta il puntatore di questo secondo utente a NULL per poter essere inizializzato di nuovo
			}
			//le istruzioni del while finiscono e legge di nuovo una riga, se e' di nuovo uguale a "Libro_prestato:" vuol dire che e' presente un altro libro e dunque la lista L_prestito guadagnera' un elemento, altrimenti verra' aggiunta la struct U alla ListaU
		}
		inserisciListaU(head,U);//Ultimo inserimento che altrimenti verrebbe saltato
	}

	fclose(f);
}

//Lettura Lista Libri
void letturaListaLibri(ListaLibri** head){
	FILE* f;
	Libro L;
	int flag=0;

	f=fopen("ListaLibri.txt","r");

	while (	fgets(L.Titolo,Max*2,f)!=NULL){
		leva(L.Titolo);

		fgets(L.Autore,Max+10,f);
		leva(L.Autore);

		fscanf(f,"\n%s %s %s\n%d\n%d\n",L.G.Genere1,L.G.Genere2,L.G.Genere3,&(L.Nlibri),&(L.Codice));

		inserisciListaLibri(head,L);
	}

	fclose(f);
}

//Lettura Libri attualmente in prestito
void letturaLibriPrestati(ListaLP** head){
	FILE* f;
	Libro_prestato Lp;
	int flag=0;

	f=fopen("LibriInPrestito.txt","r");

	if (fscanf(f,"\n%s\n%d\n%d/%d/%d\n",Lp.Username,&(Lp.Codice),&(Lp.Data_reso.gg),&(Lp.Data_reso.mm),&(Lp.Data_reso.aaaa))!=EOF){

		*head=CreaListaLP(Lp);

		while (fscanf(f,"\n%s\n%d\n%d/%d/%d\n",Lp.Username,&(Lp.Codice),&(Lp.Data_reso.gg),&(Lp.Data_reso.mm),&(Lp.Data_reso.aaaa))!=EOF){
                        inserisciLP(head,Lp);
                }
        }

	fclose(f);
}

//Lettura Data
void letturaData(Data* DFile){
	FILE* fd;

	fd=fopen("DataOdiernaProgettoAA.txt","r");

	if (fscanf(fd,"%d/%d/%d",&(DFile->gg),&(DFile->mm),&(DFile->aaaa))==EOF) {
		DFile->gg=01;
		DFile->mm=06;
		DFile->aaaa=2020;
	}

	fclose(fd);
}

//Aggiungi a Storico in "a" poiché lo storico non puo' essere modificato, ma ci sono solo aggiunte
void aggiungiStorico(Libro_prestato Lp,Data odierna){
	FILE* f;

	f=fopen("StoricoLibri.txt","a");

	Lp.Data_reso.gg=odierna.gg;
	Lp.Data_reso.mm=odierna.mm;
	Lp.Data_reso.aaaa=odierna.aaaa;

	fprintf(f,"\n%s\n%d\n%d/%d/%d\n",Lp.Username,Lp.Codice,Lp.Data_reso.gg,Lp.Data_reso.mm,Lp.Data_reso.aaaa);

	fclose(f);
}

//Salvataggio su FILE
//Lista Libri
void salvaListaLibri(ListaLibri* head){
	FILE* f;
	f=fopen("ListaLibri.txt","w");

	while(head!=NULL){
		fprintf(f,"%s\n%s\n%s %s %s\n%d\n%d\n",head->L.Titolo,head->L.Autore,head->L.G.Genere1,head->L.G.Genere2,head->L.G.Genere3,head->L.Nlibri,head->L.Codice);
		head=head->next;
	}
	fclose(f);
}

//ListaU
void salvaListaU(ListaU* head){
	ListaLP* temp;

	FILE* f;
	f=fopen("ListaU.txt","w");

	while(head!=NULL){
		fprintf(f,"\n%s\n%s\n",head->user.Username,head->user.Password);
		temp=head->user.L_prestito;
		while (temp!= NULL){
			fprintf(f,"Libro_prestato:");
			fprintf(f,"\n%d\n%d/%d/%d\n",temp->LibroP.Codice,temp->LibroP.Data_reso.gg,temp->LibroP.Data_reso.mm,temp->LibroP.Data_reso.aaaa);
			temp=temp->next;
		}
		head=head->next;
	}
	fclose(f);
}

//ListaLP
void salvaListaLP(ListaLP* head){
	FILE* f;
	f=fopen("LibriInPrestito.txt","w");

	while(head!=NULL){
		fprintf(f,"\n%s\n%d\n%d/%d/%d\n",head->LibroP.Username,head->LibroP.Codice,head->LibroP.Data_reso.gg,head->LibroP.Data_reso.mm,head->LibroP.Data_reso.aaaa);
		head=head->next;
	}
	fclose(f);
}

//Data Sistema
void salvaData(Data Odierna){
	FILE* fd;
	fd=fopen("DataOdiernaProgettoAA.txt","w");

	fprintf(fd,"%d/%d/%d",Odierna.gg,Odierna.mm,Odierna.aaaa);

	fclose(fd);
}

//Aggiorna Utente
void aggiornaU(ListaU** head,Utente U){
	ListaU* temp=*head;
	while(temp!=NULL && strcmp(temp->user.Username,U.Username)!=0){//Cerco il nodo dell'utente
		temp=temp->next;
	}
	if (temp!=NULL){//E così aggiorno la lista L_prestito
		temp->user=U;
	}
}

//Stampa
//Stampa Storico da FILE
void stampaStorico(Data Start,Data End){
	FILE* st;
	Libro_prestato Lp;
	int count=0;

	st=fopen("StoricoLibri.txt","r");

        if (ComparaDate(Start,End)>0){
                printf("La Data di inzio e' maggiore della data di fine");
        } else {
                while (fscanf(st,"\n%s\n%d\n%d/%d/%d\n",Lp.Username,&(Lp.Codice),&(Lp.Data_reso.gg),&(Lp.Data_reso.mm),&(Lp.Data_reso.aaaa))!=EOF && ComparaDate(Lp.Data_reso,Start)>=0){
                        if (ComparaDate(Lp.Data_reso,End)<=0){
				if (count==0){
					fprintf(stdout,"\n\tStorico Libri dal %d/%d/%d al %d/%d/%d.\n",Start.gg,Start.mm,Start.aaaa,End.gg,End.mm,End.aaaa);
				}
                                fprintf(stdout,"\nUsername:%s\nCodice del Libro preso in prestito: %d\nRiconsegnato il:%d/%d/%d\n",Lp.Username,Lp.Codice,Lp.Data_reso.gg,Lp.Data_reso.mm,Lp.Data_reso.aaaa);
				count++;
                        }
                }
        }
	fclose(st);

	if (count==0) {
		printf("\nNon sono stati riconsegnati libri dal %d/%d/%d al %d/%d/%d.\n",Start.gg,Start.mm,Start.aaaa,End.gg,End.mm,End.aaaa);
	}
}

int main(){
	int log,scelta,log1,codice1,ris,flag=0,flagd=1;
	char sceltaData;
	char oggetto[Max*2];

	Utente UtenteLoggato;
	Utente Utente1;

	Libro Libro1;

	Libro_prestato LibroP1;

	Data End,Start,Odierna,DFile;

	ListaU* Utenti=NULL;

	ListaLibri* Libri=NULL;
	ListaLibri* Lcercato=NULL;

	ListaLP* LibriInPrestito=NULL;

	//Creiamo i FILE per la prima apertura
	creafile("ListaU.txt");
	creafile("ListaLibri.txt");
	creafile("StoricoLibri.txt");
	creafile("DataOdiernaProgettoAA.txt");

	//Lettura dei FILE
	letturaListaU(&(Utenti));
	letturaListaLibri(&(Libri));
	letturaData(&DFile);

	//INIZIALIZZAZIONE DATA ODIERNA
	do{
		printf("1)Modifica la data\n2)Non Modificare.\nOpzione: ");
		scelta=8;
		scanf("%d",&scelta);
		puliscibuffer;
		if (scelta==1){
			do{
				printf("Personalizzare la data? (Altrimenti andra' un giorno avanti)\nY/N:");
				sceltaData=getchar();
				puliscibuffer;

				if (sceltaData=='Y' || sceltaData=='y'){

					inizializzaD(&Odierna);

					//Controllo se la data immessa e' precedente all'Ultima registrata
					if (ComparaDate(Odierna,DFile)<0){
						printf("La Data inizializzata e' precedente all'ultima registrata,");
						giornopiu(&DFile);
						Odierna=DFile;
					}

				} else if (sceltaData=='N' || sceltaData=='n') {
					giornopiu(&DFile);
					Odierna=DFile;
				} else {
					printf("Immetti Y o N!\n");
				}

			} while (sceltaData!='Y' && sceltaData!='y' && sceltaData!='N' && sceltaData!='n');
		} else if (scelta==2) {
		Odierna=DFile;
		} else {
			printf("\tSelezionare 1 o 2!\n");
		}
	} while (scelta!=1 && scelta!=2);

	//Stampo la Data del Sistema
	printf("La Data del sistema e' impostata su %d/%d/%d \n",Odierna.gg,Odierna.mm,Odierna.aaaa);

do{
	//INIZIO MENU
	//Login
	do{
	printf("Login:");

	printf("\nUsername(non deve contenere spazi):\n");
        fgets(UtenteLoggato.Username,Max,stdin);
	string_check(UtenteLoggato.Username,0);

        printf("Password(non deve contenere spazi):\n");
        fgets(UtenteLoggato.Password,Max,stdin);
	string_check(UtenteLoggato.Password,0);

	log=LogU(Utenti,&(UtenteLoggato));//Controlla l'esistenza dell'utente e se esiste gli associa la corretta lista di libri in prestito
	if (log==2){
		printf("L'Utente non esiste!");
	}
	if (log==3){
		printf("La Password Inserita non e' corretta");
	}
	} while (log!=0 && log!=1);
	//Accesso come Admin
	if (log==0){
		printf("\n\tACCESSO COME AMMINISTRATORE EFFETTUATO\n");
		//inizializzo la lista LibriInPrestito (admin)
		inizializzaListaLPadmin(&(LibriInPrestito),Utenti);

		do{
			//Menu
			printf("\tMenu:");
			printf("\n\t-1)Termina Programma\n\t0)Torna al Login\n\t1)Gestione Utenti\n\t2)Gestione Biblioteca\n\tOpzione: ");
			scelta=8;
			scanf("%d",&scelta);
			puliscibuffer;
			switch (scelta){

				case 1:{do{
					printf("\n\tGESTIONE UTENTI ");
					printf("\n\t\t1)Crea Account Utente\n\t\t2)Cancella Account Utente\n\t\t3)Torna Indietro\n\t\tOpzione: ");
					scelta=8;
					scanf("%d",&scelta);
					puliscibuffer;
					switch (scelta){

						case 1:{printf("CREA ACCOUNT UTENTE\n");
							inizializzaU(&(Utente1));
							ris=inserisciListaU(&(Utenti),Utente1);
							if (ris==0){
								printf("Inserimento Effettuato.\n");
							} else if (ris==1){
								printf("L'Username e' gia' in uso.\n");
							}

							};break;

						case 2:{printf("CANCELLA ACCOUNT UTENTE\n");
							inizializzaU(&(Utente1));
							EliminaU(&(Utenti),Utente1);
							};break;
						case 3: break;
						//non si deve poter selezionare 0 da questo menu, ma solo da quello principale
						case 0:case -1:{printf("Seleziona una delle opzioni!\n");
								 scelta=-2;
								};break;
						default: printf("Seleziona una delle opzioni!\n"); break;

					}
					} while (scelta!=3);
					};break;

				case 2:{do{
					printf("\n\tGESTIONE BIBLIOTECA");
					printf("\n\t\t1)Aggiungi Libro\n\t\t2)Cancella Libro\n\t\t3)Elenco Libri\n\t\t4)Ricerca Libro\n\t\t5)Visualizza Libri Attualmente in Prestito\n\t\t6)Visualizza Storico\n\t\t7)Torna Indietro\n\t\tOpzione: ");
					scelta=8;
					scanf("%d",&scelta);
					puliscibuffer;
					switch (scelta){

						case 1:{printf("AGGIUNGI LIBRO\n");
							inizializzaL(&(Libro1),Libri);

							ris=inserisciListaLibri(&(Libri),Libro1);
							if (ris==0){
                                                               printf("Inserimento Effettuato.\n");
                                                        } else if (ris==1) {
                                                               printf("Il libro era gia' presente, di conseguenza verra' aggiornata solo la disponibilita'.\n");
                                                        }
							};break;

						case 2:{printf("CANCELLA LIBRO\n");
							//RICERCA DEL LIBRO
							printf("Libro (Ricerca per codice o per Titolo): ");
							fgets(oggetto,Max*2,stdin);
							string_check(oggetto,1);

							if (riconosci(oggetto,5)==1){
								codice1=converti(oggetto);
								Lcercato=cercaCodice(Libri,codice1);
							}else{
								Lcercato=cercaTitolo(Libri,oggetto);
							}
							//Libro salvato in Lcercato(e' il nodo precedente al nodo da eliminare)
							if ((Lcercato->next)!=NULL){
								EliminaL(&(Lcercato),&(Libri),oggetto,LibriInPrestito);
							} else if ((Lcercato->next)==NULL){
								printf("Il Libro non e' presente.\n");
							}
							};break;

						case 3:{printf("ELENCO LIBRI\n");
							if (Libri==NULL){
								printf("\nNon sono presenti Libri.\n");
							}
							stampaListaLibri(Libri);
							};break;

						case 4:{printf("RICERCA LIBRO\n");
							//RICERCA DEL LIBRO
							printf("Libro (Ricerca per codice o per Titolo): ");
                                                        fgets(oggetto,Max*2,stdin);
							string_check(oggetto,1);

                                                        if (riconosci(oggetto,5)==1){//La stringa e' un codice
                                                               codice1=converti(oggetto);
                                                               Lcercato=cercaCodice(Libri,codice1);

								if (Lcercato!=Libri){//Controllo che il libro trovato non sia in testa alla lista
									if ((Lcercato->next)==NULL){
										printf("Nessun Libro trovato.\n");
										flag=0;
									}else {
										Libro1=(Lcercato->next)->L;
										flag=1;
									}
								} else{//Se il nodo e' la testa della lista
									if (Lcercato==NULL){
										printf("Non ci sono libri.\n");
										flag=0;
									} else {
										if (strcmp(oggetto,Lcercato->L.Titolo)==0){
											Libro1=Lcercato->L;
										} else {
											Libro1=(Lcercato->next)->L;
										}
											flag=1;
									}
								}
							} else {//La stringa immessa e' un titolo
								FiltraT(Libri,oggetto);
								flag=0;
							}
							//Se la flag e' uguale ad 1 il libro e' stato trovato e salvato su Libro1
							if (flag==1){stampaL(Libro1);}
							};break;

						case 5:{printf("VISUALIZZA TUTTI I LIBRI ATTUALMENTE IN PRESTITO\n");
							if (LibriInPrestito==NULL){
                                                                printf("\nNon sono presenti Libri.\n");
                                                        }
							stampaListaLP(LibriInPrestito);
							};break;

						case 6:{printf("STAMPA STORICO\nSelezionare periodo di tempo:\n");
							printf("\tDal:\n");
							inizializzaD(&(Start));

							printf("\n\tAl:\n");
							inizializzaD(&(End));

							stampaStorico(Start,End);

							};break;

						case 7: break;
						//non si deve poter selezionare 0 o da questo menu, ma solo da quello principale
						case 0:case -1:{ printf("Seleziona una delle opzioni!\n");
								 scelta=-2;
								};break;
						default: printf("Seleziona una delle opzioni!\n"); break;
					}
					}while (scelta!=7);
					};break;

				case 0:case -1: break;
				default: printf("\nSeleziona una delle opzioni disponibili!\n");
				}
		} while(scelta!=0 && scelta!=-1);
	}

	//Accesso come Utente
	if (log==1){
		printf("\t %s Accesso Effettuato\n",UtenteLoggato.Username);

		do{
			//Menu
			printf("\tMenu:");
			printf("\n\t-1)Termina il Programma\n\t0)Torna al Login\n\t1)Elenco Libri\n\t2)Ricerca Libro\n\t3)Visualizza Libri da Restituire\n\t4)Richiesta Libro\n\t5)Resa Libro\n\tOpzione: ");
			scelta=8;
			scanf("%d",&scelta);
			puliscibuffer;
			switch (scelta){

				case 1:{printf("ELENCO LIBRI\n");
					if (Libri==NULL){
                                 	       printf("\nNon sono presenti Libri.\n");
                                        }
					stampaListaLibri(Libri);
					};break;

				case 2:{do{
					printf("\tRICERCA o FILTRA\n");
					printf("\t\t1)Ricerca Libro\n\t\t2)Filtra Libri per Genere\n\t\t3)Filtra Libri per Autore\n\t\t4)Indietro\n\t\tOpzione: ");
					scelta=8;
					scanf("%d",&scelta);
					puliscibuffer;

					switch (scelta) {
						case 1:{//RICERCA DEL LIBRO
							printf("Libro (Ricerca per codice o per Titolo): ");
                                                        fgets(oggetto,Max*2,stdin);
							string_check(oggetto,1);

                                                        if (riconosci(oggetto,5)==1){//La stringa e' un codice
                                                               codice1=converti(oggetto);
                                                               Lcercato=cercaCodice(Libri,codice1);

                                                                if (Lcercato!=Libri){//Controllo che il libro trovato non sia in testa alla lis$
                                                                        if ((Lcercato->next)==NULL){
                                                                                printf("Nessun Libro trovato.\n");
                                                                                flag=0;
                                                                        }else {
                                                                                Libro1=(Lcercato->next)->L;
                                                                                flag=1;
                                                                        }
                                                                } else{//Se il nodo e' la testa della lista
                                                                        if (Lcercato==NULL){
                                                                                printf("Non ci sono libri.\n");
                                                                                flag=0;
                                                                        } else {
                                                                                if (strcmp(oggetto,Lcercato->L.Titolo)==0){
                                                                                        Libro1=Lcercato->L;
                                                                                } else {
                                                                                        Libro1=(Lcercato->next)->L;
                                                                                }
										flag=1;
                                                                        }
                                                                }
                                                        } else {//La stringa immessa e' un titolo
                                                                FiltraT(Libri,oggetto);
                                                                flag=0;
                                                        }
							//Se la flag e' uguale ad 1 il libro e' stato trovato e salvato su Libro1
                                                        if (flag==1){
								stampaL(Libro1);
							}
							scelta=4;
							};break;

						case 2:{printf("FILTRA PER GENERE.\nGenere: ");
							fgets(oggetto,Max,stdin);
							string_check(oggetto,0);

							FiltraG(Libri,oggetto);
							scelta=4;
							};break;

						case 3:{printf("FILTRA PER AUTORE.\nAutore: ");
							fgets(oggetto,Max+10,stdin);
							string_check(oggetto,1);

							FiltraA(Libri,oggetto);
							scelta=4;
							};break;

						case 4: break;

						default:{printf("Seleziona una delle opzioni!\n");
							 scelta=-2;
							};break;

					}
					} while (scelta!=4);
					};break;
				case 3:{printf("LISTA LIBRI DA RESTITUIRE\n");
					if (UtenteLoggato.L_prestito==NULL){
                 	                       printf("\nNon sono presenti Libri.\n");
                                        }
					stampaListaLP(UtenteLoggato.L_prestito);
					};break;

				case 4:{printf("RICHIESTA LIBRO\n");
					//RICERCA DEL LIBRO
					printf("Libro (Ricerca per codice o per Titolo): ");
                                        fgets(oggetto,Max*2,stdin);
					string_check(oggetto,1);

                                        if (riconosci(oggetto,5)==1){//La stringa e' un codice
                                                 codice1=converti(oggetto);
                                                 Lcercato=cercaCodice(Libri,codice1);
					} else {
						Lcercato=cercaTitolo(Libri,oggetto);
					}
					if (Lcercato!=Libri){//Il nodo non e' la testa della lista                                                                                                                  }
					     if ((Lcercato->next)==NULL){
                               		                printf("Nessun Libro trovato.\n");
							flag=0;
                                                }else {
                                                        Libro1=(Lcercato->next)->L;
							flag=1;
                                                }
                                        } else{//Se il nodo e' la testa della lista
                                                if (Lcercato==NULL){
                                                        printf("Non ci sono libri.\n");
							flag=0;
                                                } else {
                                                        if (strcmp(oggetto,Lcercato->L.Titolo)==0){
	                                                        Libro1=Lcercato->L;
                                                        } else {
                                                                Libro1=(Lcercato->next)->L;
                                                        }
                                                        flag=1;
                                                }
                                        }
					//Se la flag e' uguale ad 1 il libro e' stato trovato e salvato su Libro1
                                        if (flag==1){
						stampaL(Libro1);

						if (Libro1.Nlibri==0){
							printf("Libro Non Piu' Disponibile\n");
						}else {//Se il libro e' disponibile inizializzo la struct Libro_prestato, aggiungo il libro alla lista
                                                        inizializzaLP(&(LibroP1),UtenteLoggato.Username,Libro1.Codice,Odierna);
							inserisciLP(&(UtenteLoggato.L_prestito),LibroP1);
							//Diminuisco la disponibilita' del libro
							cambiadisp(&(Libri),Libro1,0);
							printf("\nIl Libro e' stato preso in prestito.\n");
						}
                                        }

					};break;

				case 5:{printf("RESA LIBRO\n");
					//RICERCA DEL LIBRO
					printf("Libro da Riconsegnare (Ricerca per codice o per Titolo): ");
                                        fgets(oggetto,Max*2,stdin);
					string_check(oggetto,1);

					//Cerco il libro nella lista dei libri per averne il codice
                                        if (riconosci(oggetto,5)==1){
                                               	codice1=converti(oggetto);
                                                Lcercato=cercaCodice(Libri,codice1);
                                        }else{
                                                Lcercato=cercaTitolo(Libri,oggetto);
                                        }
                                        if (Lcercato!=Libri){//Controllo che il libro trovato non sia in testa alla lis$
                                                if ((Lcercato->next)==NULL){
                                               	        printf("Nessun Libro trovato.\n");
							flag=0;
                                                }else {
                                                        Libro1=(Lcercato->next)->L;
							flag=1;
                                                }
                                        } else{//Se il nodo e' la testa della lista
                                                if (Lcercato==NULL){
                                                        printf("Non ci sono libri.\n");
							flag=0;
                                                } else {
                                                        if (strcmp(oggetto,Lcercato->L.Titolo)==0){
                                                               	Libro1=Lcercato->L;
                                                        } else {
                                                                Libro1=(Lcercato->next)->L;
                                                        }
                                                        flag=1;
                                                }
                                        }
					//Se la flag e' uguale ad 1 il libro e' stato trovato e salvato come Libro1
                                        if (flag==1){//Se il libro esiste allora lo elimino (nella funzione EliminaLP ci sono gia' i controlli sull'esistenza del libro in prestito e sulla sua associazione all'UtenteLoggato)
						stampaL(Libro1);
						EliminaLP(&(UtenteLoggato.L_prestito),UtenteLoggato.Username,Libro1.Codice,Odierna);

						//Aumento la disponibilita' del Libro appena riconsegnato
						cambiadisp(&(Libri),Libro1,1);

						//Aggiunge il libro appena riconsegnato allo storico
						inizializzaLP(&(LibroP1),UtenteLoggato.Username,Libro1.Codice,Odierna);
						aggiungiStorico(LibroP1,Odierna);
					}

					};break;

				case 0: case -1:{//Aggiorno l'utente nella lista
					aggiornaU(&(Utenti),UtenteLoggato);
					}; break;

				default:printf("Seleziona una delle Opzioni!\n");break;
			}

		} while(scelta!=0 && scelta!=-1);
	}

} while (scelta!=-1);

	//Savlataggio Stato del Programma
	salvaListaU(Utenti);
	salvaListaLibri(Libri);
	salvaData(Odierna);

}
