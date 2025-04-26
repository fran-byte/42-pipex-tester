#!/bin/bash

# =============================================
# TESTS PARA PIPEX (4 argumentos: infile cmd1 cmd2 outfile)
# =============================================

# Configuración
INFILE="infile"          # Archivo de entrada de prueba
OUTFILE="outfile_pipex"  # Salida de tu pipex
OUTFILE_BASH="outfile_bash"  # Salida del pipe real
TESTFILE="testfile.txt"  # Archivo temporal para tests

# Crear archivo de prueba
echo -e "hola mundo\npipe testing\n123" > $INFILE
echo -e "esto es un test\nlinea 2\nfin" > $TESTFILE

# ---- Tests básicos ----
echo -e "\n\033[1;36m>>> Tests básicos:\033[0m"

# 1. Contar líneas
./pipex $INFILE "cat" "wc -l" $OUTFILE
< $INFILE cat | wc -l > $OUTFILE_BASH
diff $OUTFILE $OUTFILE_BASH && echo "✅ Test 1 (wc -l) OK" || echo "❌ Test 1 FAIL"

# 2. Buscar texto y contar palabras
./pipex $INFILE "grep pipe" "wc -w" $OUTFILE
< $INFILE grep pipe | wc -w > $OUTFILE_BASH
diff $OUTFILE $OUTFILE_BASH && echo "✅ Test 2 (grep + wc) OK" || echo "❌ Test 2 FAIL"

# ---- Tests con rutas absolutas ----
echo -e "\n\033[1;36m>>> Tests con rutas absolutas:\033[0m"

./pipex $INFILE "/bin/cat" "/usr/bin/wc -c" $OUTFILE
< $INFILE /bin/cat | /usr/bin/wc -c > $OUTFILE_BASH
diff $OUTFILE $OUTFILE_BASH && echo "✅ Test 3 (rutas absolutas) OK" || echo "❌ Test 3 FAIL"

# ---- Tests de errores ----
echo -e "\n\033[1;36m>>> Tests de manejo de errores:\033[0m"

# 4. Comando 1 no existe
./pipex $INFILE "no_existo" "wc -l" $OUTFILE 2>/dev/null
[ $? -ne 0 ] && echo "✅ Test 4 (cmd1 error) OK" || echo "❌ Test 4 FAIL"

# 5. Comando 2 no existe
./pipex $INFILE "cat" "no_existo" $OUTFILE 2>/dev/null
[ $? -ne 0 ] && echo "✅ Test 5 (cmd2 error) OK" || echo "❌ Test 5 FAIL"

# 6. Archivo de entrada no existe
./pipex "no_existo.txt" "cat" "wc -l" $OUTFILE 2>/dev/null
[ $? -ne 0 ] && echo "✅ Test 6 (infile error) OK" || echo "❌ Test 6 FAIL"

# ---- Tests avanzados ----
echo -e "\n\033[1;36m>>> Tests avanzados:\033[0m"

# 7. Transformación de texto (mayúsculas)
./pipex $INFILE "tr a-z A-Z" "cat" $OUTFILE
< $INFILE tr a-z A-Z | cat > $OUTFILE_BASH
diff $OUTFILE $OUTFILE_BASH && echo "✅ Test 7 (tr + cat) OK" || echo "❌ Test 7 FAIL"

# 8. Procesamiento múltiple (grep + sort + head)
./pipex $TESTFILE "grep 'e'" "sort" $OUTFILE
< $TESTFILE grep 'e' | sort > $OUTFILE_BASH
diff $OUTFILE $OUTFILE_BASH && echo "✅ Test 8 (grep + sort) OK" || echo "❌ Test 8 FAIL"

# ---- Limpieza ----
rm -f $INFILE $TESTFILE $OUTFILE $OUTFILE_BASH

echo -e "\n\033[1;32mTests completados!\033[0m"
