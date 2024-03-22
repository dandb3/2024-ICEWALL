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
  - 만약 존재한다면, loadable segment entry보다 앞에 있어야 한다.
- PT_LOPROC, PT_HIPROC
- PT_GNU_STACK

##### 
#####
#####
#####
#####
#####
#####