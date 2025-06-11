# sistoperativos

void heavy_computation(int seconds, size_t mem_mb) {
    // Convertir MB a bytes
    size_t mem_size = mem_mb * 1024 * 1024;

    // Reservar memoria
    char *memory_block = malloc(mem_size);
    if (memory_block == NULL) {
        perror("No se pudo alocar memoria");
        return;
    }

    // Rellenar memoria para evitar optimizaciones del sistema
    for (size_t i = 0; i < mem_size; i++) {
        memory_block[i] = (char)(i % 256);
    }

    clock_t start = clock();
    while ((clock() - start) / CLOCKS_PER_SEC < seconds) {
        // Acceso aleatorio a la memoria para mantenerla en uso
        for (size_t i = 0; i < mem_size; i += 4096) {  // paso de 4KB (una página)
            memory_block[i] = (char)((memory_block[i] + 1) % 256);
        }
    }

    free(memory_block);
}

int main(int argc, char *argv[]) {
    int rank, size;
    int stress_time = 30;      // Tiempo por defecto
    size_t mem_mb = 100;       // Memoria por defecto (100 MB)

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    if (argc > 1) {
        stress_time = atoi(argv[1]);
    }
    if (argc > 2) {
        mem_mb = atoi(argv[2]);
    }

    printf("Proceso %d de %d iniciando carga de %d segundos con %zu MB de RAM.\n", rank, size, stress_time, mem_mb);
    fflush(stdout);

    heavy_computation(stress_time, mem_mb);

    printf("Proceso %d finalizó su carga.\n", rank);
    fflush(stdout);

    MPI_Finalize();
    return 0;
}
