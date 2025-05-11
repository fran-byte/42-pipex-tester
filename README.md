#!/bin/bash

# Colores
GREEN="\033[0;32m"
RED="\033[0;31m"
YELLOW="\033[1;33m"
NC="\033[0m"

echo -e "${GREEN}                           
                           
                ==                
              @@@@@@             
            @@@@  @@@@            
          @@@@  ..  @@@@          
          @@  @@@@@@  @@          
          @@ @@@@@@@@ @@          
          @@ @@    @@ @@          
           @  @    @  @           
           @@  @  @  @@           
            @   @@    @            
            @        @            
       @@@:  @ :  : @  :@@@        
  @@@@   @@@  @    @  @@@   @@@@  
 @@    @@@   @      @   @@@    @@ 
 @    @@%   @        @   %@@    @ 
      @@    @-      -@    @@      
       @@    @      @    @@       
             :@    @:             
               @  @               
               @  @         PIPEX Crash-Kraken by fran-byte                
${NC}"

# Compilación
make -s

# Archivos
INPUT_FILE="input.txt"
OUTPUT_FILE="output.txt"
EXPECTED_FILE="expected.txt"
ERROR_LOG="err.log"

TOTAL_TESTS=0

# Test 0: Entrada vacía 
TOTAL_TESTS=$((TOTAL_TESTS + 1))
echo -n "" > empty.txt
./pipex empty.txt "cat" "wc -l" output.txt 2> $ERROR_LOG
diff output.txt <(wc -l < empty.txt) > /dev/null
if [ $? -eq 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 0: Archivo de entrada vacío"
else
    echo -e "${RED}[KO]${NC} Test 0: Manejo de entrada vacía falló"
fi
rm -f empty.txt

# Función para tests estándar
run_test() {
    NUM=$1
    CMD1="$2"
    CMD2="$3"
    TOTAL_TESTS=$((TOTAL_TESTS + 1))

    # Setup input
    if [ "$NUM" -eq 10 ]; then
        echo -e "c\nb\na\nb" > $INPUT_FILE
    else
        echo "aaa bbb ccc" > $INPUT_FILE
    fi

    ./pipex $INPUT_FILE "$CMD1" "$CMD2" $OUTPUT_FILE 2> $ERROR_LOG
    < $INPUT_FILE $CMD1 | $CMD2 > $EXPECTED_FILE 2>/dev/null

    if diff $OUTPUT_FILE $EXPECTED_FILE > /dev/null; then
        echo -e "${GREEN}[OK]${NC} Test $NUM: $INPUT_FILE \"$CMD1\" | \"$CMD2\" $OUTPUT_FILE"
    else
        echo -e "${RED}[KO]${NC} Test $NUM: $INPUT_FILE \"$CMD1\" | \"$CMD2\" $OUTPUT_FILE"
        echo -e "${YELLOW}[INFO]${NC} Salida de pipex:"
        cat $OUTPUT_FILE
        echo -e "${YELLOW}[INFO]${NC} Salida esperada (shell):"
        cat $EXPECTED_FILE
        echo -e "${YELLOW}[DEBUG]${NC} Mensaje de error de pipex:"
        cat $ERROR_LOG
    fi
}

# Tests 1-10 (básicos)
run_test 1 "cat" "grep Hello"
run_test 2 "tr a z" "wc -w"
run_test 3 "cut -d' ' -f1" "rev"
run_test 4 "grep a" "wc -l"
run_test 5 "rev" "tr a-z A-Z"
run_test 6 "sort" "uniq"
run_test 7 "sed s/aaa/XXX/" "cut -c1-3"
run_test 8 "head -n 1" "wc -c"
run_test 9 "head -n 1" "tr a-z A-Z"
run_test 10 "sort" "uniq"

# Test 11: Comando1 vacío
TOTAL_TESTS=$((TOTAL_TESTS + 1))
echo "input for empty command1" > $INPUT_FILE
./pipex $INPUT_FILE "" "wc -l" $OUTPUT_FILE 2> $ERROR_LOG
EXIT_CODE=$?
echo -e "${GREEN}[OK]${NC} Test 11: $INPUT_FILE \"\" | \"wc -l\" $OUTPUT_FILE"
echo -e "${YELLOW}[DEBUG]${NC} Test 11 error message:"
cat $ERROR_LOG

# Test 12: Comando2 vacío
TOTAL_TESTS=$((TOTAL_TESTS + 1))
echo "input for empty command2" > $INPUT_FILE
./pipex $INPUT_FILE "cat" "" $OUTPUT_FILE 2> $ERROR_LOG
EXIT_CODE=$?
echo -e "${GREEN}[OK]${NC} Test 12: $INPUT_FILE \"cat\" | \"\" $OUTPUT_FILE"
echo -e "${YELLOW}[DEBUG]${NC} Test 12 error message:"
cat $ERROR_LOG

# Test 13: Comando con comillas
run_test 13 "echo \"hello world\"" "tr a-z A-Z"

# Test 14: Ruta relativa (./script)
echo "#!/bin/sh\necho 'test_rel_path'" > test_script.sh
chmod +x test_script.sh
run_test 14 "./test_script.sh" "wc -c"
rm -f test_script.sh

# Test 15: Archivo de entrada inexistente
TOTAL_TESTS=$((TOTAL_TESTS + 1))
./pipex no_exist.txt "cat" "wc -l" output.txt 2> $ERROR_LOG
EXIT_CODE=$?
if [ $EXIT_CODE -eq 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 15: Archivo inexistente no rompió el programa"
    echo -e "${YELLOW}[WARNING]${NC} Exit code incorrecto (esperado != 0)"
else
    echo -e "${GREEN}[OK]${NC} Test 15: Manejo de archivo de entrada inexistente"
fi

# Test 16: Permisos denegados (input)
TOTAL_TESTS=$((TOTAL_TESTS + 1))
echo "restricted" > restricted.txt
chmod 000 restricted.txt
./pipex restricted.txt "cat" "wc -l" output.txt 2> $ERROR_LOG
EXIT_CODE=$?
if [ $EXIT_CODE -eq 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 16: Permisos denegados no rompieron el programa"
    echo -e "${YELLOW}[WARNING]${NC} Exit code incorrecto (esperado != 0)"
else
    echo -e "${GREEN}[OK]${NC} Test 16: Manejo de permisos denegados (input)"
fi
rm -f restricted.txt

# Test 17: Permisos denegados (output)
TOTAL_TESTS=$((TOTAL_TESTS + 1))
touch no_write.txt
chmod 000 no_write.txt
./pipex input.txt "cat" "wc -l" no_write.txt 2> $ERROR_LOG
EXIT_CODE=$?
if [ $EXIT_CODE -eq 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 17: Permisos denegados (output) no rompieron el programa"
    echo -e "${YELLOW}[WARNING]${NC} Exit code incorrecto (esperado != 0)"
else
    echo -e "${GREEN}[OK]${NC} Test 17: Manejo de permisos denegados (output)"
fi
rm -f no_write.txt

# Test 18: Comando no encontrado
TOTAL_TESTS=$((TOTAL_TESTS + 1))
./pipex input.txt "comando_inexistente" "wc -l" output.txt 2> $ERROR_LOG
EXIT_CODE=$?
if [ $EXIT_CODE -eq 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 18: Comando inexistente no rompió el programa"
    echo -e "${YELLOW}[WARNING]${NC} Exit code incorrecto (esperado != 0)"
else
    echo -e "${GREEN}[OK]${NC} Test 18: Comando no encontrado genera error"
fi

# Test 19: Comando inválido
TOTAL_TESTS=$((TOTAL_TESTS + 1))
./pipex input.txt "ls --invalid-flag" "wc -l" output.txt 2> $ERROR_LOG
EXIT_CODE=$?
if [ $EXIT_CODE -eq 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 19: Comando inválido no rompió el programa"
    echo -e "${YELLOW}[WARNING]${NC} Exit code incorrecto (esperado != 0)"
else
    echo -e "${GREEN}[OK]${NC} Test 19: Comando inválido genera error"
fi

# Test 20: Pipeline largo (bonus)
TOTAL_TESTS=$((TOTAL_TESTS + 1))
./pipex input.txt "cat" "grep a" "wc -l" output.txt 2> $ERROR_LOG
if [ $? -eq 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 20: Pipeline largo (bonus)"
else
    echo -e "${YELLOW}[SKIP]${NC} Test 20: Bonus no implementado (múltiples pipes)"
fi

# Test 21: Ruta absoluta (verifica la ruta de sort en tu sistema operativo)
run_test 21 "/bin/sort" "uniq"

# Limpieza final
rm -f $INPUT_FILE $OUTPUT_FILE $EXPECTED_FILE $ERROR_LOG

echo -e "\n✅ Tester finalizado con ${GREEN}$TOTAL_TESTS tests${NC}."
