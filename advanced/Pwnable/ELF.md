# ELF
- Executable and Linking Format
- 실행파일, 오브젝트 파일, 라이브러리 파일 등이 취하고 있는 일종의 형식.
- 프로그램이 실행될 때, 이 포맷을 바탕으로 메모리에 매핑이 된다.

## 1. 기본 타입
```c
ElfN_Addr       Unsigned program address, uintN_t
ElfN_Off        Unsigned file offset, uintN_t
ElfN_Section    Unsigned section index, uint16_t
ElfN_Versym     Unsigned version symbol information, uint16_t
Elf_Byte        unsigned char
ElfN_Half       uint16_t
ElfN_Sword      int32_t
ElfN_Word       uint32_t
ElfN_Sxword     int64_t
ElfN_Xword      uint64_t
```

## 2. 기본 구성 요소
- ELF header
  - 기본적인 이 파일(ELF)에 대한 정보들이 저장되어 있다.
- Program header table
  - 각 세그먼트 별 정보가 저장되어 있다.
  - 특히, 실행 시에 메모리에 어떻게 매핑되는지에 대한 정보가 저장되어 있다.
- Section header table
  - 각 섹션 별 정보가 저장되어 있다.

### 2.1. ELF Header
- 현재 ELF 파일에 대한 여러 정보들이 담겨있는 header이다.

#### 2.1.1. 기본 구조
```c
#define EI_NIDENT 16

typedef struct {
    unsigned char e_ident[EI_NIDENT];
    uint16_t      e_type;
    uint16_t      e_machine;
    uint32_t      e_version;
    ElfN_Addr     e_entry;
    ElfN_Off      e_phoff;
    ElfN_Off      e_shoff;
    uint32_t      e_flags;
    uint16_t      e_ehsize;
    uint16_t      e_phentsize;
    uint16_t      e_phnum;
    uint16_t      e_shentsize;
    uint16_t      e_shnum;
    uint16_t      e_shstrndx;
} ElfN_Ehdr;
```

##### e_ident
16바이트의 크기로 이루어져 있다.
파일을 어떤 식으로 해석해야 하는지에 대한 정보가 담겨 있다.
- 0x00 ~ 0x03
  - 파일의 매직 넘버
  - 해당 파일이 ELF 포맷을 가지고 있음을 나타낸다.
  - ELF 파일의 경우 ```\x7f\x45\x4c\x46``` 으로 이루어져 있다.
- 0x04
  - binary의 아키텍쳐 정보
  - 32비트 / 64비트
- 0x05
  - 데이터 인코딩 방식
  - little endian / big endian
- 0x06
  - ELF의 버전 정보
  - 아직은 따로 버전이 나누어져 있지 않음.
- 0x07
  - 대상으로 하는 os나 abi에 대한 정보
  - System V, Solaris, Linux ... 등등이 있다.
- 0x08
  - 위에 정해진 ABI의 구체적인 버전 정보
- 0x09 ~ 0x0f
  - 바이트 패딩을 위한 부분
  - 0으로 초기화된다.

##### e_type
현재 파일의 타입을 나타낸다.
- relocatable file
- executable file
- shared object
- core file

##### e_machine
요구되는 아키텍쳐에 대한 정보가 담겨있다.
- AMD x86-64, Intel 80386 ...

##### e_version
파일 버전? 을 의미한다.

##### e_entry
프로그램의 시작 지점의 virtual address를 의미
만약 시작 지점이 없다면 0으로 세팅

##### e_phoff
현재 파일에서 program header table의 시작지점 offset을 의미
program header table이 없다면 0으로 세팅.

##### e_shoff
현재 파일에서 section header table의 시작지점 offset을 의미
section header table이 없다면 0으로 세팅.

##### e_flags
processor-specific 플래그 값들이 저장된다.

##### e_ehsize
ELF header의 사이즈를 나타낸다.

##### e_phentsize
program header table의 한 entry의 사이즈

##### e_phnum
program header table의 총 entry 수

##### e_shentsize
section header table의 한 entry 사이즈

##### e_shnum
section header table의 총 entry 수

##### e_shstrndx
각 section의 name에 대한 string을 담고 있는 section name string table이 있는데, 이 table 또한 하나의 section으로써 존재한다.
이 때, 이 table이 위치해 있는 section header table에서의 index 값을 의미한다.

#### 2.1.2. 실제 hexdump 결과


### 2.2. Program Header Table
- 각 세그먼트들에 대한 정보가 쓰여있는 테이블이다.
- 각 세그먼트의 메모리 매핑 위치, 권한, 등에 대한 정보가 담겨있음.
- 파일 상 위치 : ELF header의 e_phoff에 적혀있는 offset부터 시작한다.

#### 2.2.1. 기본 구조

program header table의 각 entry들은 다음과 같은 구조를 가진다.
```c
typedef struct {
    uint32_t   p_type;
    Elf32_Off  p_offset;
    Elf32_Addr p_vaddr;
    Elf32_Addr p_paddr;
    uint32_t   p_filesz;
    uint32_t   p_memsz;
    uint32_t   p_flags;
    uint32_t   p_align;
} Elf32_Phdr;

typedef struct {
    uint32_t   p_type;
    uint32_t   p_flags;
    Elf64_Off  p_offset;
    Elf64_Addr p_vaddr;
    Elf64_Addr p_paddr;
    uint64_t   p_filesz;
    uint64_t   p_memsz;
    uint64_t   p_align;
} Elf64_Phdr;
```

##### p_type
어떤 종류의 세그먼트인지, 혹은 어떤 식으로 해석해야 하는지에 대한 정보가 저장된다.
- PT_NULL
  - 현재 entry는 사용되지 않음을 의미한다.
- PT_LOAD
  - loadable segment를 의미한다. (메모리에 매핑될 수 있다는 뜻)
  - p_filesz, p_memsz, p_vaddr에 의해 메모리에 어떻게 매핑될 수 있는지가 정해진다.
  - PT_LOAD 세그먼트끼리는 p_vaddr의 값을 기준으로 오름차순으로 정렬된다.
- PT_DYNAMIC
  - dynamic linking information을 담고 있음을 의미함.
- PT_INTERP
  - interpreter로 호출할 null-terminated pathname에 대한 위치와 사이즈가 저장된다.
  - 만약 존재한다면, loadable segment entry보다 앞에 있어야 한다.
- PT_NOTE
  - notes의 위치를 의미한다. (ElfN_Nhdr)
- PT_SHLIB
  - ...
- PT_PHDR
  - program header table 자기 자신에 대한 정보가 담겨있다.
  - location과 size에 대한 정보가 담겨있음.
  - program header table이 memory image의 일부일 경우에만 쓰인다.
  - 만약 존재한다면, loadable segment entry보다 앞에 있어야 한다.
- PT_LOPROC, PT_HIPROC
  - [PT_LOPROC, PT_HIPROC]의 범위에 있는 값들은 processor-specific하게 reserved 되어있다.
- PT_GNU_STACK
  - p_flags 멤버변수에 있는 값을 토대로 stack의 상태를 조정하는 데에 쓰이는 GNU extension이다.
  - Linux kernel에 의해 사용된다.

##### p_offset
segment의 첫 바이트가 파일의 처음으로부터 얼마나 떨어져 있는지 (offset)의 값.

##### p_vaddr
segment의 첫 바이트가 메모리상에서 어떤 virtual address에서 시작하는지의 값.

##### p_paddr
physical address가 사용되는 경우 사용된다. p_vaddr와 의미상 동등하다.

##### p_filesz
segment의 file image의 크기를 나타낸다.

##### p_memsz
segment의 memory image의 크기를 나타낸다.

##### p_flags
해당 segment의 권한 설정을 나타낸다.
- PF_X : executable
- PF_W : writable
- PF_R : readable

##### p_align
loadable process segment는 p_vaddr와 p_offset을 page size로 나눈 나머지가 동일해야 함.
0이나 1은 alignment가 필요없다는 뜻이다.
나머지 경우 : p_align은 양수이면서, 2의 제곱수이면서, p_vaddr와 p_offset은 p_align으로 나눈 나머지가 동일해야 한다.

### 2.3. Section header table
- 각 section들에 대한 정보가 쓰여있는 table이다.
- initial entry, SHN_LORESERVE 와 SHN_HIRESERVE 사이의 index들은 reserved 되어있다.
  - initial entry : e_phnum, e_shnum, e_shstrndx에 사용되고, 이 외에는 0으로 설정된다.
- object file의 경우 special indices가 없다.
- 파일 상 위치 : ELF header의 e_shoff에 적혀있는 offset부터 시작한다.

#### 2.3.1. 기본 구조
```c
           typedef struct {
               uint32_t   sh_name;
               uint32_t   sh_type;
               uint32_t   sh_flags;
               Elf32_Addr sh_addr;
               Elf32_Off  sh_offset;
               uint32_t   sh_size;
               uint32_t   sh_link;
               uint32_t   sh_info;
               uint32_t   sh_addralign;
               uint32_t   sh_entsize;
           } Elf32_Shdr;

           typedef struct {
               uint32_t   sh_name;
               uint32_t   sh_type;
               uint64_t   sh_flags;
               Elf64_Addr sh_addr;
               Elf64_Off  sh_offset;
               uint64_t   sh_size;
               uint32_t   sh_link;
               uint32_t   sh_info;
               uint64_t   sh_addralign;
               uint64_t   sh_entsize;
           } Elf64_Shdr;
```

##### sh_name
section의 name을 의미하는 멤버변수이다.
section header string table에서의 index 값이 저장된다.

##### sh_type
section의 내용과 구성이 어떻게 되어있는지 알려준다.

- SHT_NULL
  - section header가 inactive 한 상태임을 나타낸다.
- SHT_PROGBITS
  - 프로그램에 의해 정의된 정보가 담긴다.
  - 정보가 저장되는 방식 또한 프로그램에 의해 정의되어 있다.
- SHT_SYMTAB
  - symbol table을 담고 있다.
  - 
  
### 2.4. string table
null-terminated character sequence (string)를 담고 있다.
symbol이나 section name을 표시하기 위해 쓰인다.
첫 byte (index zero)는 null byte, 마지막 byte 또한 null byte이다.

### 2.5. symbol table
각 symbol의 이름, 종류, 크기 위치 등의 정보들이 저장되어 있는 테이블이다.

#### 2.5.1. 기본 구조
```c
           typedef struct {
               uint32_t      st_name;
               Elf32_Addr    st_value;
               uint32_t      st_size;
               unsigned char st_info;
               unsigned char st_other;
               uint16_t      st_shndx;
           } Elf32_Sym;

           typedef struct {
               uint32_t      st_name;
               unsigned char st_info;
               unsigned char st_other;
               uint16_t      st_shndx;
               Elf64_Addr    st_value;
               uint64_t      st_size;
           } Elf64_Sym;
```
##### st_name
symbol string table의 index 값이 저장된다.
값이 0이 아니라면, string table index를 통해서 symbol의 이름을 찾을 수 있다.
-> string table의 0번째 index의 값이 null인 이유.

##### st_value
symbol의 값이 저장된다.

##### st_size
symbol의 size값이 저장된다.
size가 없거나 unknown 이라면 0이 저장된다.

##### st_info
symbol의 type과 binding 특성이 저장된다. (둘 중 하나가 아니라 둘 다 저장된다.)

- type
  - STT_NOTYPE
    - type이 정의되지 않았다.
  - STT_OBJECT
    - data object와 연관되어 있다.
  - STT_FUNC
    - function 혹은 executable code와 연관되어 있다.
  - STT_SECTION
    - section과 연관되어 있다.
    - 이 타입의 심볼은 주로 relocation을 위한 것이고, STB_LOCAL 바인딩을 가지고 있다..?
  - STT_FILE
    - ...
  _ STT_LOPROC, STT_HIPROC
    - [STT_LOPROC, STT_HIPROC] are reserved for processor-specific semantics.

- binding
  - STB_LOCAL
    - object file의 외부에서는 보이지 않는다.
    - 그래서 같은 이름일지라도 서로 간섭하지 않고 존재할 수 있다.
  - STB_GLOBAL
    - 모든 object files에서 보인다.
    - 따로 정의되어 있지 않은 다른 object file에서 참조 가능.
  - STB_WEAK
    - global symbol과 비슷하지만, 우선순위가 더 낮다.
  - STB_LOPROC, STB_HIPROC
    - [STB_LOPROC, STB_HIPROC] are reserved for processor-specific semantics.

아래의 매크로들로 binding과 type field를 추출해낼 수 있다.
- ELF32_ST_BIND(info), ELF64_ST_BIND(info)
  - st_info로부터 binding을 추출한다.
- ELF32_ST_TYPE(info), ELF64_ST_TYPE(info)
  - st_info로부터 type을 추출한다.
- ELF32_ST_INFO(bind, type), ELF64_ST_INFO(bind, type)
  - binding과 type으로부터 st_info를 만들어준다.
  
##### st_other
symbol의 visibility를 정의한다.

- STV_DEFAULT
  - Default symbol visibility rules.
  - Global과 weak symbol들은 다른 module에게 모두 보인다.
  - local module 에서의 참조는 다른 modules에서의 정의된 값으로 될 수 있다.
- STV_INTERNAL
  - processor-specific hidden class
- STV_HIDDEN
  - 다른 module에게 symbol이 보이지 않는다.
  - local module에서의 참조는 local symbol만 가능하다.
- STV_PROTECTED
  - symbol이 다른 module에게도 보이지만, local module에서의 참조는 오직 local symbol에 의해서만 이루어진다.
  
-> STV_DEFAULT와 STV_PROTECTED의 차이 : 당연히 local symbol은 볼 수 없다. 하지만 global / weak symbol은 둘 다 볼 수 있는데, 대신 참조가 preempt되지 않는다. 즉, 같은 이름의 다른 module에서 정의된 2개의 전역변수가 있는 경우, 현재 module에서 정의된 전역변수만을 참조한다는 뜻이다.

visibility type을 추출하는 매크로
- ELF32_ST_VISIBILITY(other)
- ELF64_ST_VISIBILITY(other)

##### st_shndx
모든 symbol table entry는 몇 개의 section과 연관되어 있다.
이 연관된 section header index 값이 저장된다.

### 2.6. relocation entries (Rel & Rela)

#### 2.6.1. 기본 구조
- addend가 필요하지 않은 Relocation structure
```c
           typedef struct {
               Elf32_Addr r_offset;
               uint32_t   r_info;
           } Elf32_Rel;

           typedef struct {
               Elf64_Addr r_offset;
               uint64_t   r_info;
           } Elf64_Rel;
```

- addend가 필요한 Relocation structure
```c
           typedef struct {
               Elf32_Addr r_offset;
               uint32_t   r_info;
               int32_t    r_addend;
           } Elf32_Rela;

           typedef struct {
               Elf64_Addr r_offset;
               uint64_t   r_info;
               int64_t    r_addend;
           } Elf64_Rela;
```

##### r_offset
relocation action을 취해야 할 location을 의미한다.

- relocatable file의 경우
  - section의 시작점부터 relocation으로 인한 storage unit까지의 offset을 의미한다.
- executable file / shared object 의 경우
  - relocation으로 인한 storage unit의 virtual address을 의미한다.
  
##### r_info

##### r_addend
https://velog.io/@junttang/SP-7.1-Fundamentals-of-Linking
https://refspecs.linuxbase.org/elf/gabi4+/ch4.reloc.html
https://deepfield.blog/kr/ctf/basic/elf%20%ED%8C%8C%EC%9D%BC%20%ED%98%95%EC%8B%9D%EC%97%90%EC%84%9C%20%EC%9E%AC%EB%B0%B0%EC%B9%98(relocation),%20%EB%A7%81%ED%82%B9(linking)%20%EA%B9%8C%EC%A7%80/