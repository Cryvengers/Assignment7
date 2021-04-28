# Toy version of SHA-3
This is the decryption of toy version of [SHA-3](https://hello.iitk.ac.in/sites/default/files/cs641a2021/resources/NIST.FIPS_.202_0.pdf) with b=1600 bit state matrix. In this there are only three step mappings namely theta, chi, pi and they follow the following algorithm for encrption. However this toy version is not same as general SHA-3 encryption.
```cpp
#include <iostream>
using namespace std
int main(){
        char str[str_length];
	char hexa[16] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'};

	uint64_t b = 1600;
	uint64_t l = 512;
	uint64_t c = 1024;
	uint64_t r = 576;
	int rounds = 24;
	int i, j, k;


	uint64_t state[5][5][64], tempstate[5][5][64];

	for(i = 0; i < 5; ++i)
		for(j = 0; j < 5; ++j)
			for(k = 0; k < 64; ++k)
				state[i][j][k] = 0;

	uint64_t message[r];

	k = 0;
	for(i = 0; i < str_length; ++i){
		for(j = 7; j >= 0; --j){
			if(((str[i] >> j) & 1) == 1)
				message[k] = 1;
			else
				message[k] = 0;
			++k;
		}
	}
	while(k < r){
		message[k] = 0;
		++k;
	}
	for(k = 0; k < r; ++k)
		state[k/(64*5)][(k/64) % 5][k%64] = message[k];
	uint64_t current_round = 0;
	uint64_t column_parity[5][64];
	while(current_round < rounds){
		//theta operation
		for(i = 0; i < 5; ++i){
			for(k = 0; k < 64; ++k){
				column_parity[i][k] = 0;
				for(j = 0; j < 5; ++j)
					column_parity[i][k] ^= state[i][j][k];
			}
		}

		for(i = 0; i < 5; ++i){
			for(j = 0; j < 5; ++j){
				for(k = 0; k < 64; ++k){
					state[i][j][k] ^= column_parity[(i+4)%5][k] ^ column_parity[(i+1)%5][k];
					tempstate[i][j][k] = state[i][j][k];
				}
			}
		}

		//pi operation
		for(i = 0; i < 5; ++i)
			for(j = 0; j < 5; ++j)
				for(k = 0; k < 64; ++k)
					state[j][((2 * i) + (3 * j)) % 5][k] = tempstate[i][j][k];
		

		//chi operation
		for(i = 0; i < 5; ++i)
			for(j = 0; j < 5; ++j)
				for(k = 0; k < 64; ++k)
					tempstate[i][j][k] = state[i][j][k];


		for(i = 0; i < 5; ++i)
			for(j = 0; j < 5; ++j)
				for(k = 0; k < 64; ++k)
					state[i][j][k] = tempstate[i][j][k] ^ (~tempstate[i][(j+1)%5][k] & tempstate[i][(j+2)%5][k]);

		++current_round;
	}
	
	uint64_t index;

	k = 0;
	while(k < l){

		index = 0;
		for(j = 3; j >= 0; --j)
			index = index*2 + state[k/(64*5)][(k/64)%5][k%64 + j];
		printf("%c", hexa[index]);
		k += 4;
		if(k == 256)
			printf("\n\t");
	}
}
  ```
We decrypted the hash value that we were given and was told that the input is atmost 16 bytes.
Given hash value is ```6E626264000000000188808065E16CE26FEEEBEC65E16CE2018C898865E16CE20004090800000000018C898865E16CE26E666B6C000000006E62626400000000```.
As per code, the encryption is taking place in the order theta,pi,chi and for 24 rounds. 
Some observations are that there are only 128 bits and so there are only 2 non-zero blocks along 3rd dimension and remaining blocks are all 0's, the operations taking place are only along first and second and nothing along third.
Lets suppose first 64 bits as $`m_1`$
