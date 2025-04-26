# pipex-little-test


# =============================================
# PRUEBAS BÁSICAS (Funcionalidad principal)
# =============================================

# 1. Pipe básico (contar líneas)
./pipex file1.txt "cat" "wc -l" file2.txt          # Equivale a: < file1.txt cat | wc -l > file2.txt

# 2. Comandos con argumentos
./pipex file1.txt "grep 'hola'" "awk '{print \$1}'" file2.txt  # Filtra y extrae la primera columna

# 3. Rutas absolutas de comandos
./pipex file1.txt "/bin/cat" "/usr/bin/wc -w" file2.txt        # Cuenta palabras usando rutas completas

# =============================================
# PRUEBAS DE ERRORES (Robustez)
# =============================================

# 4. Archivo de entrada inexistente
./pipex no_existe.txt "cat" "wc -l" file2.txt      # Debe fallar con "No such file..."

# 5. Comando inválido
./pipex file1.txt "comando_falso" "wc -l" file2.txt # Debe mostrar "command not found"

# 6. Permisos denegados en output
touch file_prohibido.txt && chmod 000 file_prohibido.txt
./pipex file1.txt "cat" "wc -l" file_prohibido.txt  # Debe dar "Permission denied"

# 7. Directorio como output
mkdir carpeta_falsa
./pipex file1.txt "cat" "wc -l" carpeta_falsa      # Debe fallar (es directorio)

# =============================================
# PRUEBAS AVANZADAS (Casos complejos)
# =============================================

# 8. Múltiples argumentos en comandos
./pipex file1.txt "ls -la" "grep '.txt'" file2.txt # Filtra archivos .txt del ls

# 9. Comandos anidados (usando xargs)
./pipex file1.txt "echo \"hola\"" "xargs -I {} echo {} mundo" file2.txt  # Debe imprimir "hola mundo"

# 10. Variables de entorno
./pipex file1.txt "echo \$HOME" "wc -c" file2.txt  # Cuenta caracteres de la ruta $HOME

# =============================================
# VERIFICACIÓN DE RESULTADOS
# =============================================
echo -e "\n\033[1;36m=== Resultados ===\033[0m"
cat file2.txt 2>/dev/null || echo "El archivo de salida no se creó (comportamiento esperado en pruebas de error)"