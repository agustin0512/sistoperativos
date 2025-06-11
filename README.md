#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    int rank, size;
    int seconds = 30;         // Tiempo por defecto
    size_t mem_mb = 100;      // Memoria por defecto en MB

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    if (argc > 1) seconds = atoi(argv[1]);
    if (argc > 2) mem_mb = atoi(argv[2]);

    printf("Proceso %d: Ejecutando %d segundos con %zu MB de RAM\n", rank, seconds, mem_mb);

    // Reservar memoria
    char *mem = malloc(mem_mb * 1024 * 1024);
    if (!mem) {
        perror("malloc");
        MPI_Finalize();
        return 1;
    }

    // Usar CPU y RAM
    time_t end = time(NULL) + seconds;
    while (time(NULL) < end) {
        for (size_t i = 0; i < mem_mb * 1024 * 1024; i += 4096)
            mem[i] = (char)(i % 256);  // Acceso periódico
    }

    free(mem);
    printf("Proceso %d: Terminado\n", rank);

    MPI_Finalize();
    return 0;
}
