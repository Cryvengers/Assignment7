# Toy version of SHA-3
This is the decryption of toy version of [SHA-3](https://hello.iitk.ac.in/sites/default/files/cs641a2021/resources/NIST.FIPS_.202_0.pdf) with b=1600 bit state matrix. 
In this there are only three step mappings namely theta, chi, pi and they follow the following algorithm for encrption. However this toy version is not same as general SHA-3 encryption.
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
We decrypted the hash value that we were given and was told that the input is atmost 16 bytes.\
Given hash value is ```6E626264000000000188808065E16CE26FEEEBEC65E16CE2018C898865E16CE20004090800000000018C898865E16CE26E666B6C000000006E62626400000000```.\
As per code, the encryption is taking place in the order theta,pi,chi and for 24 rounds. \
Some observations are that there are only 128 bits and so there are only 2 non-zero blocks along 3rd dimension and remaining blocks are all 0's, the operations taking place are only along first and second and nothing along third.\
Lets suppose first 64 bits as M and next 64 bits as N.\
Now all operation results will be in terms of M and N. So any block at a given round will be a direct function of M and N. So changing a bit of 3rd dimensional index k of M doesn't change other index bits.\
So we first do brute force on first 4 bits of M and N and find the pair for which we get the output O such that the first four bits of every block of O match with the first four bits of every block of the final state matrix of the hash value given. After obtaining first 4 we repeat this for the next four and we continue till we get all the bits by the below code.
```python
import numpy as np

hexa = ['0', '1', '2', '3', '4', '5', '6', '7', 
        '8', '9', 'A', 'B', 'C', 'D', 'E', 'F']
output = "6E626264000000000188808065E16CE26FEEEBEC65E16CE2018C898865E16CE20004090800000000018C898865E16CE26E666B6C000000006E62626400000000"

rem = 15 #substitute values from 0-15 to get m1, m2 in sequence
b = 1600
l = 512
c = 1024
r = 576
rounds = 24

state = np.zeros((5,5,64))
tempstate = np.zeros((5,5,64))
four_bits = ['0000', '0001', '0010', '0011',
             '0100', '0101', '0110', '0111',
             '1000', '1001', '1010', '1011',
             '1100', '1101', '1110', '1111']

given_str = ''
for i in range(rem,len(output),16):
    given_str += output[i]
print(given_str)

for m2 in four_bits:
    for m1 in four_bits:
        
        message = '0000' * rem + m1 + '0' * (60 - rem*4) + 
        '0000' * rem + m2 + '0' * (60 - rem*4) + '0' * 448

        for i in range(5):
            for j in range(5):
                for k in range(64):
                    state[i,j,k] = 0

        for k in range(r):
            state[k//(64*5), (k//64)%5, k%64] = message[k]


        current_round = 0
        column_parity = np.zeros((5,64))


        while (current_round < rounds):

            #theta operation
            for i in range(5):
                for k in range(64):
                    column_parity[i, k] = 0
                    for j in range(5):
                        column_parity[i, k] = int(column_parity[i, k]) ^ int(state[i, j, k])

            for i in range(5):
                for j in range(5):
                    for k in range(64):
                        state[i, j, k] = int(state[i, j, k]) ^ (int(column_parity[(i+4)%5, k]) ^ int(column_parity[(i+1)%5, k]))
                        tempstate[i, j, k] = state[i, j, k]


            #pi operation
            for i in range(5):
                for j in range(5):
                    for k in range(64):
                        state[j, ((2 * i) + (3 * j)) % 5, k] = tempstate[i, j, k]


            #chi operation
            for i in range(5):
                for j in range(5):
                    for k in range(64):
                        tempstate[i, j, k] = state[i, j, k]

            for i in range(5):
                for j in range(5):
                    for k in range(64):
                        state[i, j, k] = int(tempstate[i, j, k]) ^ int(~int(tempstate[i, (j+1)%5, k]) & int(tempstate[i, (j+2)%5, k]))

            current_round += 1


        k = 0
        out_str = ''
        while (k < l):

            index = 0
            for j in [3,2,1,0]:
                index = index * 2 + state[k//(64*5), (k//64) % 5, (k % 64 + j)]
            if k%64 == 4*rem:
                out_str += hexa[int(index)]
            k += 4
        
        if out_str == given_str:
            print(out_str)
            print(m1)
            print(m2)
            break
    else:
        continue
    break
```
