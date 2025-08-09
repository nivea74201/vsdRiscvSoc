🧩 Task 2: Prove Your Local RISC‑V Setup (Run, Disassemble, Decode)

🎯Objective

    ✅ Run 4 RISC‑V C programs locally using the installed toolchain and spike pk
    ✅ Embed uniqueness via username, hostname, machine ID, and timestamps
    ✅ Disassemble and decode main section of each binary
    ✅ Decode RISC-V integer instructions manually
    
🧩 Task 2.1 - Set Up Unique Identity Variables

🎯 Objective

Set identity variables in the Linux host shell so that each build is uniquely tied to the system.

⚙️ Commands Used

   export U=$(id -un)
   export H=$(hostname -s)
   export M=$(cat /etc/machine-id | head -c 16)
   export T=$(date -u +%Y-%m-%dT%H:%M:%SZ)
   export E=$(date +%s)
   
✅ Summary
   .Stored username, hostname, machine ID, UTC time, and epoch time as environment variables
   .These values are passed as #define macros to every program
   
🔴 Output

<img width="778" height="158" alt="Screenshot from 2025-08-10 00-35-25" src="https://github.com/user-attachments/assets/7b689318-40a5-491b-bd37-8f8a963a6a2d" />

🧩 Task 2.2 - Create Common Header unique.h

🎯 Objective

Create a reusable header for printing build/run metadata like user, host, machine ID, build time, etc.

⚙️ Common Header code

#ifndef UNIQUE_H
#define UNIQUE_H
#include <stdio.h>
#include <stdint.h>
#include <time.h>
#ifndef USERNAME
#define USERNAME "unknown_user"
#endif
#ifndef HOSTNAME
#define HOSTNAME "unknown_host"
#endif
#ifndef MACHINE_ID
#define MACHINE_ID "unknown_machine"
#endif
#ifndef BUILD_UTC
#define BUILD_UTC "unknown_time"
#endif
#ifndef BUILD_EPOCH
#define BUILD_EPOCH 0
#endif
static uint64_t fnv1a64(const char *s) {
const uint64_t OFF = 1469598103934665603ULL, PRIME = 1099511628211ULL;
uint64_t h = OFF;
for (const unsigned char *p=(const unsigned char*)s; *p; ++p) {
h ^= *p; h *= PRIME;
}
return h;
}
static void uniq_print_header(const char *program_name) {
time_t now = time(NULL);
char buf[512];
int n = snprintf(buf, sizeof(buf), "%s|%s|%s|%s|%ld|%s|%s",
USERNAME, HOSTNAME, MACHINE_ID, BUILD_UTC,
(long)BUILD_EPOCH, __VERSION__, program_name);
(void)n;
uint64_t proof = fnv1a64(buf);
char rbuf[600];
snprintf(rbuf, sizeof(rbuf), "%s|run_epoch=%ld", buf, (long)now);
uint64_t runid = fnv1a64(rbuf);
printf("=== RISC-V Proof Header ===\n");
printf("User : %s\n", USERNAME);
printf("Host : %s\n", HOSTNAME);
printf("MachineID : %s\n", MACHINE_ID);
printf("BuildUTC : %s\n", BUILD_UTC);
printf("BuildEpoch : %ld\n", (long)BUILD_EPOCH);
printf("GCC : %s\n", __VERSION__);
printf("PointerBits: %d\n", (int)(8*(int)sizeof(void*)));
printf("Program : %s\n", program_name);
printf("ProofID : 0x%016llx\n", (unsigned long long)proof);
printf("RunID : 0x%016llx\n", (unsigned long long)runid);
printf("===========================\n");
}
#endifContains:

    Preprocessor macros
    fnv1a64() hash function
    uniq_print_header() function that prints a unique proof block

✅ Summary

    Generates ProofID (compile-unique) and RunID (per-execution unique)

🧩 Task 2.3 - Program 1: factorial.c

🎯 Objective

Run a recursive factorial calculation while embedding unique metadata

⚙ Factorial Code

#include "unique.h"
static unsigned long long fact(unsigned n){ return (n<2)?1ULL:n*fact(n-1); }
int main(void){
uniq_print_header("factorial");
unsigned n = 12;
printf("n=%u, n!=%llu\n", n, fact(n));
return 0;
}

⚙ Compile Command

riscv64-unknown-elf-gcc -O0 -g -march=rv64imac -mabi=lp64 \
-DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
-DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
factorial.c -o factorial

▶ Run

spike pk ./factorial

🔴 Output of spike

<img width="463" height="277" alt="Screenshot from 2025-08-09 22-15-57" src="https://github.com/user-attachments/assets/7009863b-38eb-4015-876d-cec92e06af2c" />

🧠 Assembly
 
 riscv64-unknown-elf-gcc -O0 -S factorial.c -o factorial.s

🛠 Disassemble Main

riscv64-unknown-elf-objdump -d ./factorial | sed -n '/<main>:/,/^$/p' | tee factorial_main_objdump.txt
<img width="404" height="534" alt="Screenshot from 2025-08-09 00-33-19" src="https://github.com/user-attachments/assets/a866e7f0-483b-47ff-b781-1743e47f941d" />
<img width="768" height="497" alt="Screenshot from 2025-08-09 00-38-20" src="https://github.com/user-attachments/assets/22198a8b-03d5-4383-b70e-4c46c949e8f0" />

🧩 Task 2.4 - Program 2: max_array.c

🎯 Objective

Find the maximum in an array and print with proof header

⚙ Max Array Code

#include "unique.h"
int main(void){
uniq_print_header("max_array");
int a[] = {42,-7,19,88,3,88,5,-100,37};
int n = sizeof(a)/sizeof(a[0]), max=a[0];
for(int i=1;i<n;i++) if(a[i]>max) max=a[i];
printf("Array length=%d, Max=%d\n", n, max);
return 0;
}
(Repeat same steps as Task 2.3 for compile, run, assembly, and disassembly)

🔴 Output of spike

<img width="463" height="277" alt="Screenshot from 2025-08-09 22-31-58" src="https://github.com/user-attachments/assets/4824d7e8-0445-4cde-a9c9-392c088e50a7" />

🔴 Output

<img width="405" height="493" alt="Screenshot from 2025-08-09 22-35-48" src="https://github.com/user-attachments/assets/6009ac27-5e4b-48ed-89d7-a711cc6a7c34" />
<img width="804" height="679" alt="Screenshot from 2025-08-09 22-36-45" src="https://github.com/user-attachments/assets/2e3b476e-6904-4b76-a6bb-6e34cc85b920" />

🧩 Task 2.5 - Program 3: bitops.c

🎯 Objective

Perform basic bitwise operations and show uniqueness

⚙ Bitops Code

#include "unique.h"
int main(void){
uniq_print_header("bitops");
unsigned x=0xA5A5A5A5u, y=0x0F0F1234u;
printf("x&y=0x%08X\n", x&y);
printf("x|y=0x%08X\n", x|y);
printf("x^y=0x%08X\n", x^y);
printf("x<<3=0x%08X\n", x<<3);
printf("y>>2=0x%08X\n", y>>2);
return 0;
}
(Repeat same steps as Task 2.3)

🔴 Output of spike
<img width="360" height="351" alt="Screenshot from 2025-08-09 22-40-29" src="https://github.com/user-attachments/assets/77a9846c-1e7f-4009-a094-41a055c34f71" />

🔴 Output
<img width="1075" height="686" alt="Screenshot from 2025-08-09 22-47-59" src="https://github.com/user-attachments/assets/c1de174c-c18f-4b19-bab3-920d353b4295" />
<img width="1075" height="686" alt="Screenshot from 2025-08-09 22-43-25" src="https://github.com/user-attachments/assets/240dabcd-ffbe-4bd8-acea-f3b10e2b8ccc" />
<img width="1081" height="604" alt="Screenshot from 2025-08-09 22-48-18" src="https://github.com/user-attachments/assets/6e61c724-8eae-431f-aaad-fa8fcb92c6df" />

🧩 Task 2.6 - Program 4: bubble_sort.c

🎯 Objective

Perform bubble sort and print sorted array with proof header

⚙ Bubble sort Code

#include "unique.h"
void bubble(int *a,int n){ for(int i=0;i<n-1;i++) for(int j=0;j<n-1-i;j++) if(a[j]>a[j
+1]){int t=a[j];a[j]=a[j+1];a[j+1]=t;} }
int main(void){
uniq_print_header("bubble_sort");
int a[]={9,4,1,7,3,8,2,6,5}, n=sizeof(a)/sizeof(a[0]);
bubble(a,n);
printf("Sorted:"); for(int i=0;i<n;i++) printf(" %d",a[i]); puts("");
return 0;
}
(Repeat same steps as Task 2.3)

🔴 Output of spike

<img width="523" height="297" alt="Screenshot from 2025-08-09 22-49-56" src="https://github.com/user-attachments/assets/5e2772c1-8fdb-43b5-af20-88af10db009a" />

🔴 Output

<img width="523" height="587" alt="Screenshot from 2025-08-09 22-50-48" src="https://github.com/user-attachments/assets/004957ee-b770-49dc-a8ca-2d74336539c8" />

<img width="833" height="694" alt="Screenshot from 2025-08-09 22-51-40" src="https://github.com/user-attachments/assets/60a45c4e-76c0-4d03-aecb-d5ad231c1a19" />

🧩 Task 2.7 - Instruction Decoding

🎯 Objective

Manually decode at least 5 RISC‑V integer instructions from .s or .objdump output. 
The detailed instruction decoding for all programs can be found here:
FACTORIAL

| Address | Machine Code (Hex) | Machine Code (Binary)            | Format | Opcode  | rd       | rs1      | rs2      | funct3 | funct7 / imm | Description / Meaning                       |
| ------- | ------------------ | -------------------------------- | ------ | ------- | -------- | -------- | -------- | ------ | ------------ | ------------------------------------------- |
| 10378   | 78078513           | 01111000000001111000010100010011 | I-type | 0010011 | x10 (a0) | x15 (a5) | —        | 000    | imm = 1920   | a0 = a5 + 1920 (addi)                       |
| 10382   | fef42623           | 11111110111101000010011000100011 | S-type | 0100011 | —        | x8 (s0)  | x15 (a5) | 010    | imm = -20    | Mem\[s0 - 20] = a5 (sw)                     |
| 10386   | fec42783           | 11111110110001000010011110000011 | I-type | 0000011 | x15 (a5) | x8 (s0)  | —        | 010    | imm = -20    | a5 = Mem\[s0 - 20] (lw)                     |
| 1039c   | 79078513           | 01111001000001111000010100010011 | I-type | 0010011 | x10 (a0) | x15 (a5) | —        | 000    | imm = 1936   | a0 = a5 + 1936 (addi)                       |
| 103a0   | 1ac000ef           | 00011010110000000000000011101111 | J-type | 1101111 | x1 (ra)  | —        | —        | —      | imm = 428    | ra = PC+4; jump to PC + 428 (jal to printf) |

MAX_ARRAY
| Address | Machine Code (Hex) | Machine Code (Binary)            | Format | Opcode  | rd       | rs1      | rs2      | funct3 | funct7 / imm | Description / Meaning   |
| ------- | ------------------ | -------------------------------- | ------ | ------- | -------- | -------- | -------- | ------ | ------------ | ----------------------- |
| 10332   | 7b078513           | 01111011000001111000010100010011 | I-type | 0010011 | x10 (a0) | x15 (a5) | —        | 000    | imm = 1968   | a0 = a5 + 1968 (addi)   |
| 1033c   | 7e078793           | 01111110000001111000011110010011 | I-type | 0010011 | x15 (a5) | x15 (a5) | —        | 000    | imm = 2016   | a5 = a5 + 2016 (addi)   |
| 10348   | fcb43023           | 11111100101101000011000000100011 | S-type | 0100011 | —        | x8 (s0)  | x11 (a1) | 011    | imm = -64    | Mem\[s0 - 64] = a1 (sd) |
| 1035a   | fef42023           | 11111110111101000010000000100011 | S-type | 0100011 | —        | x8 (s0)  | x15 (a5) | 010    | imm = -32    | Mem\[s0 - 32] = a5 (sw) |
| 1037a   | ff040713           | 11111111000001000000011100010011 | I-type | 0010011 | x14 (a4) | x8 (s0)  | —        | 000    | imm = -16    | a4 = s0 - 16 (addi)     |

BITOPS
| Address | Machine Code (Hex) | Machine Code (Binary)            | Format | Opcode  | rd       | rs1      | rs2      | funct3 | funct7 / imm     | Description / Meaning   |
| ------- | ------------------ | -------------------------------- | ------ | ------- | -------- | -------- | -------- | ------ | ---------------- | ----------------------- |
| 10332   | 7a078513           | 01111010000001111000010100010011 | I-type | 0010011 | x10 (a0) | x15 (a5) | —        | 000    | imm = 1952       | a0 = a5 + 1952 (addi)   |
| 1033e   | 5a578793           | 01011010010101111000011110010011 | I-type | 0010011 | x15 (a5) | x15 (a5) | —        | 000    | imm = 1445       | a5 = a5 + 1445 (addi)   |
| 10342   | fef42623           | 11111110111101000010011000100011 | S-type | 0100011 | —        | x8 (s0)  | x15 (a5) | 010    | imm = -20        | Mem\[s0 - 20] = a5 (sw) |
| 1035a   | 00e7f7b3           | 00000000111001111111011110110011 | R-type | 0110011 | x15 (a5) | x15 (a5) | x14 (a4) | 111    | funct7 = 0000000 | a5 = a5 AND a4 (and)    |
| 1038a   | 00e7c7b3           | 00000000111001111100011110110011 | R-type | 0110011 | x15 (a5) | x15 (a5) | x14 (a4) | 100    | funct7 = 0000000 | a5 = a5 XOR a4 (xor)    |

BUBBLE SORT
| Address | Machine Code (Hex) | Machine Code (Binary)            | Format | Opcode  | rd       | rs1      | rs2      | funct3 | funct7 / imm | Description / Meaning                     |
| ------- | ------------------ | -------------------------------- | ------ | ------- | -------- | -------- | -------- | ------ | ------------ | ----------------------------------------- |
| 10406   | 87078513           | 10000111000001111000010100010011 | I-type | 0010011 | x10 (a0) | x15 (a5) | —        | 000    | imm = -1936  | a0 = a5 + (-1936) (addi)                  |
| 10410   | 89878793           | 10001001100001111000011110010011 | I-type | 0010011 | x15 (a5) | x15 (a5) | —        | 000    | imm = -1896  | a5 = a5 + (-1896) (addi)                  |
| 1041c   | fcb43023           | 11111100101101000011000000100011 | S-type | 0100011 | —        | x8 (s0)  | x11 (a1) | 011    | imm = -64    | Mem\[s0 - 64] = a1 (sd)                   |
| 1043c   | fc040793           | 11111100000001000000011110010011 | I-type | 0010011 | x15 (a5) | x8 (s0)  | —        | 000    | imm = -64    | a5 = s0 + (-64) (addi)                    |
| 10456   | a025               | 00000000000000001010000000100101 | B-type | 1100011 | —        | x0       | x0       | 000    | imm = 0x28   | Unconditional jump to address 0x1047e (j) |
