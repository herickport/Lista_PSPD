# Lista de Exercícios PSPD: OpenMP + MPI

## Aluno: Hérick Ferreira de Souza Portugues - 18/0033034

## Questões

### Questão 1:

Para a resolução da questão 1, o tamanho do fractal `N` foi dividido em blocos, de modo que cada processo ficou responsável por preencher o vetor `pixel_array` da sua parte do bloco. Após isso, foi utilizada a função `MPI_File_write_ordered` para fazer a escrita no arquivo de forma serial e obedecendo a ordem do rank do processo. A função `MPI_File_write_ordered` foi escolhida por se tratar de uma função bloqueante (serial) e que possui ponteiro para o arquivo compartilhado entre os processos. 

O arquivo com a questão encontra-se em: [fractal_mpi_serial](mpi_serial/fractalmpiserial.c).

#### Uso

1. Acessar a pasta do arquivo
```
cd mpi_serial/
```

2. Compilar:
```
mpicc fractalmpiserial.c -o fractalmpiserial -lm
```

3. Rodar:
```
mpirun -host <LISTA_DE_HOSTS> -np <QTD_PROCESSOS> ./fractalmpiserial <TAM_FRACTAL>
```

Após a execução será gerada uma imagem com o nome: `out_julia_mpi_serial`.

### Questão 2:

Para a resolução da questão 2, manteve-se a mesma divisão do fractal por blocos, com cada processo ficando responsável por preencher o vetor `pixel_array` da sua parte do bloco. A mudança em relação a questão anterior se deu na parte da escrita do arquivo, visto que gravação deveria acontecer em paralelo. Dessa forma, para resolver a questão foi calculado o offset para cada processo, e com o offset calculado foram utilizadas as funções MPI para escrita de arquivos que se utilizam de offsets explícitos. Como cada processo poderia atuar de forma independente, foram testadas as funções `noncollective`, optando-se pela `MPI_File_iwrite_at` visto que é não bloqueante.

O arquivo com a questão encontra-se em: [fractal_mpi_io](mpi_io/fractalmpi_io.c).

**Obs:** Também é possível testar a versão bloqueante de escrita com a função `MPI_File_write_at` que também se utiliza de offset explícito. Para isso, basta comentar a linha 149 e descomentar a linha 152, no arquivo da questão. 

#### Uso

1. Acessar a pasta do arquivo
```
cd mpi_io/
```

2. Compilar:
```
mpicc fractalmpi_io.c -o fractalmpi_io -lm
```

3. Rodar:
```
mpirun -host <LISTA_DE_HOSTS> -np <QTD_PROCESSOS> ./fractalmpi_io <TAM_FRACTAL>
```

Após a execução será gerada uma imagem com o nome: `out_julia_mpi_io`.

### Questão 3:

Para a resolução da questão 3, com OpenMP, foram utilizadas duas área paralelas no código. A primeira é responsável pela escrita no vetor `pixel_array` e se utiliza do *pragma*  `parrallel for` para computar os pixels e salvar no arquivo de forma paralela. Na segunda área paralela é feita a escrita no arquivo, separando o output em blocos, e fazendo cada thread gravar uma parte do arquivo, com o uso de offset.

O arquivo com a questão encontra-se em: [fractal_omp](omp/fractalomp.c).

#### Uso

1. Acessar a pasta do arquivo
```
cd omp/
```

2. Compilar:
```
gcc fractalomp.c -o fractalomp -fopenmp -lm
```

3. Rodar:
```
export OMP_NUM_THREADS=<QTD_THREADS>; ./fractalomp <TAM_FRACTAL>
```

Após a execução será gerada uma imagem com o nome: `out_julia_omp`.

### Questão 4:

Para a comparação entre os programas foi utilizado o cluster `chococino`, rodando a partir da máquina cm1 e com 10000 linhas no fractal. Para a obtenção do tempo de execução, foi utilizada a função `time`, calculando a média entre 3 execuções.

#### Tabela de comparação com 1 processo/thread:

| Programa           | QTD Processos/Threads | Tempo de Execução |
| ------------------ | --------------------- | ----------------- |
| fractal (original) | 1 (host cm1)          | 1m4.994s          |
| fractal_mpi_serial | 1 (host cm1)          | 1m0.625s          |
| fractal_mpi_io     | 1 (host cm1)          | 1m27.857s         |
| fractal_omp        | 1 thread cm1          | 1m3.718s          |

Rodando com apenas um processo/thread o programa com MPI serial foi o que se saiu melhor, sendo seguido de perto pelo OMP e pelo fractal original. O mais lento foi o MPI com escrita paralela, com quase 30s de diferença em relação ao programa mais rápido.

#### Tabela de comparação com 2 processos/threads:

| Programa           | QTD Processos/Threads | Tempo de Execução |
| ------------------ | --------------------- | ----------------- |
| fractal_mpi_serial | 2 (hosts cm1, cm2)    | 0m35.780s         |
| fractal_mpi_io     | 2 (hosts cm1, cm2)    | 0m51.802s         |
| fractal_omp        | 2 threads cm1         | 0m35.994s         |

Na execução com 2 processos o programa MPI serial diminuiu o tempo de exeção quase pela metade em relação ao teste anterior, e continou se saindo bem melhor do que o MPI paralelo. Além disso, o programa com OMP também melhorou bastante com 2 threads, chegando a um tempo bem próximo ao MPI serial.

#### Tabela de comparação com 4 processos/threads:

| Programa           | QTD Processos/Threads        | Tempo de Execução |
| ------------------ | ---------------------------- | ----------------- |
| fractal_mpi_serial | 4 (hosts cm1, cm2, cm3, cm4) | 0m32.160s         |
| fractal_mpi_io     | 4 (hosts cm1, cm2, cm3, cm4) | 0m27.546s         |
| fractal_omp        | 4 threads cm1                | 0m31.520s         |

Com 4 processos/threads os programas MPI serial e OpenMP abaixaram bem pouco seus tempos de execução em comparação com os testes com 2 processos/threads. Em contrapartida, o programa MPI paralelo diminui seu tempo de execução quase pela metade, superando todos os outros.

### Questão 5:

Para a questão 5, foi apenas adicionado um pragma `parallel for` no primeiro `for` da função `selection_sort`, com a intenção de paralelizar a execução do algoritmo, tornando-o mais rápido.

O arquivo com a questão encontra-se em: [selection_sort_omp](selection_sort/ordena_vetor_omp.c).
