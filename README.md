# Program-CRUD
CRUD adalah singkatan yang berasal dari Create, Read, Update, dan Delete, 
dimana keempat istilah tersebut merupakan fungsi utama yang nantinya diimplementasikan  ke dalam basis data.  

{
#include <iostream>
#include <fstream>
#include <string>
#include <limits>

using namespace std;

struct KTP{
	int pk;
	char NIK[100];
	char nama[100];
	char tgl[100];
};

int Menu();
void checkdataktp(fstream &data);
void writeData(fstream &data,int posisi,KTP &inputKTP);
int getDataSize(fstream &data);
KTP readData(fstream &data, int posisi);
void adddataktp(fstream &data);
void displaydataktp(fstream &data);
void updateRecord(fstream &data);
void deleteRecord(fstream &data);

int main(){

	fstream data;

	checkdataktp(data);
	
	int pilihan = Menu();
	char is_continue;

	enum option{CREATE = 1, READ, UDATE, DELETE, FINISH};


	while(pilihan != FINISH){
		
		switch(pilihan){
			case CREATE:
				cout << "Menambah data KTP" << endl;
				adddataktp(data);
				goto label_continue;
				break;
			case READ:
				cout << "Tampilkan data KTP" << endl;
				displaydataktp(data);
				goto label_continue;
				break;
			case UDATE:
				cout << "Ubah data KTP" << endl;
				displaydataktp(data);
				updateRecord(data);
				displaydataktp(data);
				break;
			case DELETE:
				cout << "Hapus data KTP" << endl;
				displaydataktp(data);
				deleteRecord(data);
				displaydataktp(data);
				break;
			default:
				cout << "pilihan tidak ditemukan" << endl;
				break;
		}

		label_continue:

		cout << "lanjutkan?(y/n) : ";
		cin >> is_continue;
		if((is_continue == 'y') | (is_continue == 'Y')){
			pilihan = Menu();
		}else if((is_continue == 'n') | (is_continue == 'N')){
			break;
		}else{
			goto label_continue;
		}
		
	}

	cout << "akhir dari program" << endl;

	cin.get();
	return 0;
}

int Menu(){
	int input;
	system("cls");
	cout << "\nProgram CRUD data KTP" << endl;
	cout << "=============================" << endl;
	cout << "1. Tambah data KTP" << endl;
	cout << "2. Tampilkan data KTP" << endl;
	cout << "3. Ubah data KTP" << endl;
	cout << "4. Hapus data KTP" << endl;
	cout << "5. selesai" << endl;
	cout << "=============================" << endl;
	cout << "pilih [1-5] : ";
	cin >> input;
	cin.ignore(numeric_limits<streamsize>::max(),'\n');
	return input;
}

void checkdataktp(fstream &data){
	data.open("data.bin", ios::out | ios::in | ios::binary);

	//check file ada atau tidak
	if(data.is_open()){
		cout << "database ditemukan" << endl;
	}else{
		cout << "database tidak ditemukan, buat database baru" << endl;
		data.close();
		data.open("data.bin",ios::trunc | ios::out | ios::in | ios::binary);
	}
}

void writeData(fstream &data,int posisi,KTP &inputKTP){
	data.seekp((posisi - 1)*sizeof(KTP), ios::beg);
	data.write(reinterpret_cast<char*>(&inputKTP),sizeof(KTP));
}

int getDataSize(fstream &data){
	int start, end;
	data.seekg(0,ios::beg);
	start = data.tellg();
	data.seekg(0,ios::end);
	end = data.tellg();
	return(end-start)/sizeof(KTP);
}

KTP readData(fstream &data, int posisi){
	KTP readKTP;
	data.seekg((posisi - 1)*sizeof(KTP), ios::beg);
	data.read(reinterpret_cast<char*>(&readKTP),sizeof(KTP));
	return readKTP;
}

void adddataktp(fstream &data){

	KTP inputKTP, lastKTP;

	int size = getDataSize(data);

	cout << "ukuran data : " << size << endl;

	if(size == 0){
		inputKTP.pk = 1;
	}else{
		lastKTP = readData(data,size);
		cout << "pk = " << lastKTP.pk << endl;
		inputKTP.pk = lastKTP.pk + 1;
	}

	//readData(data,size)

	cout << "nama      : ";
	fflush (stdin); gets(inputKTP.nama);
	cout << "tgl lahir : ";
	fflush (stdin); gets(inputKTP.tgl);
	cout << "NIK       : ";
	fflush (stdin); gets(inputKTP.NIK);

	writeData(data,size+1,inputKTP);
}

void displaydataktp(fstream &data){
	int size = getDataSize(data);
	KTP showKTP;
	cout<< "No.\tpk.\tNIK.\t\tNama.\t\t\tTgl Lahir." << endl;
	for (int i = 1; i <= size; i++){
		showKTP = readData(data,i);
		cout << i << "\t";
		cout << showKTP.pk << "\t";
		cout << showKTP.NIK << "\t";
		cout << showKTP.nama << "\t";
		cout << showKTP.tgl << endl;
	}
}

void updateRecord(fstream &data){
	int nomor;
	KTP updateKTP;
	cout << "pilih no: ";
	cin >> nomor;
	cin.ignore(numeric_limits<streamsize>::max(),'\n');

	updateKTP = readData(data,nomor);
	cout << "\n\n pilihan data: "<< endl;
	cout << "NIK       : " << updateKTP.NIK << endl;
	cout << "nama      : " << updateKTP.nama << endl;
	cout << "tgl lahir : " << updateKTP.tgl << endl;

	cout << "\nMerubah data: " << endl;
	cout << "NIK       : ";
	fflush (stdin); gets(updateKTP.NIK);
	cout << "Nama      : ";
	fflush (stdin); gets(updateKTP.nama);
	cout << "tgl lahir : ";
	fflush (stdin); gets(updateKTP.tgl);

	writeData(data,nomor,updateKTP);
}

void deleteRecord(fstream &data){
	int nomor,size,offset;
	KTP blankKTP, tempKTP;
	fstream dataSementara;

	size = getDataSize(data);

	cout << "hapus nomor: ";
	cin >> nomor;

	writeData(data,nomor,blankKTP);

	dataSementara.open("temp.dat", ios::trunc|ios::out|ios::in|ios::binary);

	offset = 0;

	for (int i = 1; i <= size; i++){
		tempKTP = readData(data,i);

		if(nomor != i){
			writeData(dataSementara,i - offset,tempKTP);
		}else{
			offset++;
			cout << "deleted item" << endl;
		}
	}
	size = getDataSize(dataSementara);
	data.close();
	data.open("data.bin",ios::trunc|ios::out|ios::binary);
	data.close();
	data.open("data.bin",ios::out|ios::in|ios::binary);

	for (int i = 1; i <= size; i++){
		tempKTP = readData(dataSementara,i);
		writeData(data,i,tempKTP);
	}
}
}
