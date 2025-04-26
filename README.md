# pipex-little-test

# Tests básicos
echo "=== Tests básicos ==="
./pipex "ls -l" "wc -l"              # Cuenta líneas de 'ls -l'
./pipex "cat Makefile" "grep pipex"   # Busca 'pipex' en Makefile
./pipex "echo hello" "cat -e"         # Muestra 'hello$' con $ al final

# Tests con paths absolutos
echo -e "\n=== Tests con paths absolutos ==="
./pipex "/bin/ls" "/usr/bin/wc -c"    # Usa rutas completas de binarios

# Tests con comandos que fallan
echo -e "\n=== Tests de manejo de errores ==="
./pipex "comando_inexistente" "wc -c" # Primer comando falla
./pipex "ls -l" "comando_inexistente" # Segundo comando falla

# Tests comparativos con pipe real
echo -e "\n=== Tests comparativos ==="
./pipex "ls -l" "wc -l" > output_pipex
ls -l | wc -l > output_real
echo "Diferencias con pipe real:"
diff output_pipex output_real         # No debe mostrar diferencias
rm output_pipex output_real

# Test con múltiples comandos (si tu pipex lo soporta)
echo -e "\n=== Test con múltiples pipes (si implementado) ==="
./pipex "ps aux" "grep bash" "awk '{print \$2}'" "head -n 3"  # Proceso de 3 pasos

# Tests con caracteres especiales
echo -e "\n=== Tests con caracteres especiales ==="
./pipex "echo 'hola mundo'" "tr 'a-z' 'A-Z'"   # Convertir a mayúsculas
./pipex "echo \"hola\nadios\"" "grep hola"     # Manejo de saltos de línea

# Test de rendimiento con datos grandes
echo -e "\n=== Test de rendimiento ==="
./pipex "dd if=/dev/zero bs=1M count=10 2>/dev/null" "wc -c" # Debe mostrar 10485760
