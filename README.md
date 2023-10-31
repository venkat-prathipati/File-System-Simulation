#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define DISK_SIZE 1000
#define MAX_FILE_NAME 20
#define MAX_FILES 100

typedef struct {
    char name[MAX_FILE_NAME];
    int start_block;
    int size;
} File;

typedef struct {
    int block_index;
    int size;
    File* file;
} DiskBlock;

typedef struct {
    DiskBlock blocks[DISK_SIZE];
    int fragmentation;
    int wasted_blocks;
} Disk;

File files[MAX_FILES];
int file_count = 0;
Disk disk;

void initializeDisk() {
    for (int i = 0; i < DISK_SIZE; i++) {
        disk.blocks[i].block_index = i;
        disk.blocks[i].size = 0;
        disk.blocks[i].file = NULL;
    }
    disk.fragmentation = 0;
    disk.wasted_blocks = 0;
}

void allocateFileSpace(File* file) {
    // Implement a simple contiguous allocation logic
    int available_blocks = DISK_SIZE;
    for (int i = 0; i < DISK_SIZE; i++) {
        if (disk.blocks[i].size == 0) {
            int contiguous_blocks = 1;
            while (i + contiguous_blocks < DISK_SIZE && disk.blocks[i + contiguous_blocks].size == 0) {
                contiguous_blocks++;
            }
            if (contiguous_blocks >= file->size) {
                file->start_block = i;
                for (int j = i; j < i + file->size; j++) {
                    disk.blocks[j].size = file->size;
                    disk.blocks[j].file = file;
                }
                return;
            }
            i += contiguous_blocks;
        }
    }
    disk.fragmentation += file->size - available_blocks;
}

void deleteFile(File* file) {
    for (int i = file->start_block; i < file->start_block + file->size; i++) {
        disk.blocks[i].size = 0;
        disk.blocks[i].file = NULL;
    }
}

void renameFile(File* file, const char* new_name) {
    // Implement file renaming logic
    strncpy(file->name, new_name, MAX_FILE_NAME - 1);
}

void moveFile(File* file, int new_block) {
    // Implement file moving logic
    deleteFile(file);
    file->start_block = new_block;
    allocateFileSpace(file);
}

void simulateTimeUnit(int time) {
    // Simulate file operations and changes over a time unit
    // Add, delete, rename, and move files
    // Update disk fragmentation and wasted blocks

    // For simplicity, we'll randomly add, delete, rename, and move files in this example.
    int operation = rand() % 4;
    if (operation == 0 && file_count < MAX_FILES) {
        File new_file;
        snprintf(new_file.name, sizeof(new_file.name), "File%d", file_count++);
        new_file.size = rand() % 20 + 1;  // Random file size between 1 and 20 blocks
        allocateFileSpace(&new_file);
        files[file_count - 1] = new_file;
        printf("Time %d - Added file: %s, Size: %d\n", time, new_file.name, new_file.size);
    } else if (operation == 1 && file_count > 0) {
        int index = rand() % file_count;
        File* file = &files[index];
        deleteFile(file);
        printf("Time %d - Deleted file: %s, Size: %d\n", time, file->name, file->size);
        file_count--;
        for (int i = index; i < file_count; i++) {
            files[i] = files[i + 1];
        }
    } else if (operation == 2 && file_count > 0) {
        int index = rand() % file_count;
        File* file = &files[index];
        char new_name[MAX_FILE_NAME];
        snprintf(new_name, sizeof(new_name), "NewName%d", time);
        renameFile(file, new_name);
        printf("Time %d - Renamed file: %s to %s\n", time, file->name, new_name);
    } else if (operation == 3 && file_count > 0) {
        int index = rand() % file_count;
        File* file = &files[index];
        int new_block = rand() % DISK_SIZE;
        moveFile(file, new_block);
        printf("Time %d - Moved file: %s to block %d\n", time, file->name, new_block);
    }

    // Calculate fragmentation and wasted blocks
    disk.wasted_blocks = 0;
    for (int i = 0; i < DISK_SIZE; i++) {
        if (disk.blocks[i].size == 0) {
            disk.wasted_blocks++;
        }
    }
}

void saveOutputToFile(int time) {
    FILE* outputFile = fopen("simulation_output.txt", "a");
    if (outputFile == NULL) {
        perror("Failed to open file for writing");
        exit(1);
    }

    fprintf(outputFile, "Time %d - Disk status:\n", time);
    for (int i = 0; i < DISK_SIZE; i++) {
        fprintf(outputFile, "[%d:%d] ", disk.blocks[i].block_index, disk.blocks[i].size);
    }
    fprintf(outputFile, "\nFragmentation: %d\n", disk.fragmentation);
    fprintf(outputFile, "Wasted blocks: %d\n", disk.wasted_blocks);
    fprintf(outputFile, "\n");

    fclose(outputFile);
}

int main() {
    initializeDisk();

    // Initialize random number generator with current time
    srand(time(NULL));

    int total_time = 100;  // Total simulation time units

    // Simulate the disk over time
    for (int time = 0; time <= total_time; time++) {
        simulateTimeUnit(time);

        // Save the output every 10 time units to avoid large file size
        if (time % 10 == 0) {
            saveOutputToFile(time);
        }
    }

    return 0;
}
