# Atv_Vinícios_13/11

### a) Por que o algoritmo RSA não é adequado para cifrar arquivos grandes diretamente (cite limitações de bloco e desempenho)?
O RSA possui uma limitação matemática estrita de tamanho de bloco: 
o tamanho máximo dos dados que podem ser cifrados diretamente depende do tamanho da chave 
(por exemplo, uma chave RSA de 2048 bits só consegue cifrar blocos de dados menores que 245 bytes se usar o padding adequado). 
Além disso, o RSA consome muito processamento (desempenho computacional extremamente baixo) 
em comparação aos algoritmos simétricos, tornando inviável cifrar arquivos grandes diretamente.

### b) Qual é o papel da chave simétrica (AES) e qual é o papel do par de chaves assimétricas (RSA) no esquema híbrido?
Chave simétrica (AES): É utilizada para cifrar o arquivo/conteúdo de forma rápida e eficiente, lidando com volumes grandes de dados.
Par de chaves assimétricas (RSA): 
A chave pública é utilizada exclusivamente para cifrar e proteger a chave simétrica (AES) durante o transporte, 
enquanto a chave privada correspondente serve para decifrá-la de forma segura apenas pelo destinatário legítimo.

### c) Como o remetente pode garantir também a autenticidade do arquivo enviado ao professor? Qual comando OpenSSL gera uma assinatura digital?
O remetente garante a autenticidade assinando digitalmente o arquivo (ou o hash dele) 
utilizando a sua própria chave privada RSA, permitindo que o destinatário valide a autoria usando a chave pública do remetente. 
O comando OpenSSL utilizado para gerar a assinatura digital é:

"openssl dgst -sha256 -sign privada.pem -out assinatura.sig mensagem.txt"

Parte 2:

### a) Gere um par de chaves RSA de 2048 bits (privada.pem e publica.pem) e uma chave AES aleatória de 32 bytes (chave_aes.bin):

1. Gerar a chave privada RSA de 2048 bits
openssl genrsa -out privada.pem 2048

2. Extrair a chave pública correspondente
openssl rsa -in privada.pem -pubout -out publica.pem

3. Gerar uma chave AES aleatória de 32 bytes (256 bits)
openssl rand -out chave_aes.bin 32

### b) Cifre mensagem.txt com AES-256-CBC (-pbkdf2) gerando mensagem.enc, e cifre chave_aes.bin com publica.pem (openssl pkeyutl -encrypt) gerando chave_aes.enc:

1. Cifrar o arquivo de texto com AES-256-CBC
openssl enc -aes-256-cbc -pbkdf2 -in mensagem.txt -out mensagem.enc -pass file:chave_aes.bin

2. Cifrar a chave AES utilizando a chave pública RSA
openssl pkeyutl -encrypt -in chave_aes.bin -out chave_aes.enc -pubin -inkey publica.pem

### c) Realize o processo inverso (decriptação) e comprove com openssl dgst -sha256 que o hash do arquivo restaurado é idêntico ao original:

1. Decifrar a chave AES usando a chave privada RSA
openssl pkeyutl -decrypt -in chave_aes.enc -out chave_aes_restaurada.bin -inkey privada.pem

2. Decifrar o arquivo mensagem.enc usando a chave AES recuperada
openssl enc -d -aes-256-cbc -pbkdf2 -in mensagem.enc -out mensagem_restaurada.txt -pass file:chave_aes_restaurada.bin

3. Gerar e comparar os hashes SHA-256 do arquivo original e do restaurado
openssl dgst -sha256 mensagem.txt
openssl dgst -sha256 mensagem_restaurada.txt

## CODIGO C: CRIPTOGRAFIA

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Função para executar comandos do OpenSSL via sistema
void executar_comando(const char *cmd) {
    printf("[EXECUTANDO] %s\n", cmd);
    int status = system(cmd);
    if (status != 0) {
        printf("[ERRO] Falha ao executar o comando.\n");
        exit(1);
    }
}

int main() {
    printf("=== APLICATIVO DE CRIPTOGRAFIA HIBRIDA (Criptografia e Assinatura) ===\n");

    // 1. Criar arquivo de teste mensagem.txt se não existir
    FILE *f = fopen("mensagem.txt", "w");
    if (f) {
        fprintf(f, "Trabalho Final de Seguranca de Software - Criptografia Hibrida com OpenSSL.\n");
        fclose(f);
    }

    // 2. PARTE PRATICA: Comandos OpenSSL equivalentes
    // a) Gerar par de chaves RSA e chave AES aleatória de 32 bytes
    executar_comando("openssl genrsa -out privada.pem 2048");
    executar_comando("openssl rsa -in privada.pem -pubout -out publica.pem");
    executar_comando("openssl rand -out chave_aes.bin 32");

    // b) Cifrar mensagem.txt com AES-256-CBC e cifrar a chave AES com RSA pública
    executar_comando("openssl enc -aes-256-cbc -pbkdf2 -in mensagem.txt -out mensagem.enc -pass file:chave_aes.bin");
    executar_comando("openssl pkeyutl -encrypt -in chave_aes.bin -out chave_aes.enc -pubin -inkey publica.pem");

    // c) Processo inverso (Decriptação) e verificação de integridade por Hash SHA-256
    executar_comando("openssl pkeyutl -decrypt -in chave_aes.enc -out chave_aes_restaurada.bin -inkey privada.pem");
    executar_comando("openssl enc -d -aes-256-cbc -pbkdf2 -in mensagem.enc -out mensagem_restaurada.txt -pass file:chave_aes_restaurada.bin");

    printf("\n--- Verificacao de Integridade (Hash SHA-256) ---\n");
    executar_comando("openssl dgst -sha256 mensagem.txt");
    executar_comando("openssl dgst -sha256 mensagem_restaurada.txt");

    printf("\nProcesso round-trip concluido com sucesso em C!\n");
    return 0;
}

## como executar no terminal: 

gcc main.c -o programa_cripto./programa_cripto
