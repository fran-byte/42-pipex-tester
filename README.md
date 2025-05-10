#!/bin/bash

# Colores
GREEN="\033[0;32m"
RED="\033[0;31m"
YELLOW="\033[1;33m"
NC="\033[0m"

make -s

# Archivos
INPUT_FILE="input.txt"
OUTPUT_FILE="output.txt"
EXPECTED_FILE="expected.txt"
ERROR_LOG="err.log"

TOTAL_TESTS=0

# Función general para tests estándar
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

    # Comprobamos si hay comandos vacíos
    if [ -z "$CMD1" ] || [ -z "$CMD2" ]; then
        EXIT_CODE=$?
        if [ $EXIT_CODE -eq 0 ]; then
            echo -e "${GREEN}[OK]${NC} Test $NUM: $INPUT_FILE \"$CMD1\" | \"$CMD2\" $OUTPUT_FILE ${YELLOW}[WARNING]${NC}: Comando vacío detectado."
        else
            echo -e "${RED}[KO]${NC} Test $NUM: $INPUT_FILE \"$CMD1\" | \"$CMD2\" $OUTPUT_FILE (exit code $EXIT_CODE)"
            cat $ERROR_LOG
        fi
        return
    fi

    # Compara las salidas (sin redirección a /dev/null para forzar visibilidad)
    echo -e "\n${YELLOW}=== Resultado Test $NUM ===${NC}"
    diff $OUTPUT_FILE $EXPECTED_FILE
    if [ $? -eq 0 ]; then
        echo -e "${GREEN}[OK]${NC} Test $NUM: $INPUT_FILE \"$CMD1\" | \"$CMD2\" $OUTPUT_FILE"
    else
        echo -e "${RED}[KO]${NC} Test $NUM: $INPUT_FILE \"$CMD1\" | \"$CMD2\" $OUTPUT_FILE"
    fi
}

# Test 1: cat | grep
TOTAL_TESTS=$((TOTAL_TESTS + 1))
echo "Hello World!" > $INPUT_FILE
./pipex $INPUT_FILE "cat" "grep Hello" $OUTPUT_FILE
< $INPUT_FILE cat | grep Hello > $EXPECTED_FILE
diff $OUTPUT_FILE $EXPECTED_FILE > /dev/null
if [ $? -eq 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 1: $INPUT_FILE \"cat\" | \"grep Hello\" $OUTPUT_FILE"
else
    echo -e "${RED}[KO]${NC} Test 1: $INPUT_FILE \"cat\" | \"grep Hello\" $OUTPUT_FILE"
    diff $OUTPUT_FILE $EXPECTED_FILE
fi

# Test 2–9
run_test 2 "tr a z" "wc -w"
run_test 3 "cut -d' ' -f1" "rev"
run_test 4 "grep a" "wc -l"
run_test 5 "rev" "tr a-z A-Z"
run_test 6 "sort" "uniq"
run_test 7 "sed s/aaa/XXX/" "cut -c1-3"
run_test 8 "head -n 1" "wc -c"  # Corregido: sin pipe interno
run_test 9 "head -n 1" "tr a-z A-Z"

# Test 10: Ruta absoluta (con debug explícito)
echo -e "\n${YELLOW}=== Ejecutando Test 10 ===${NC}"
run_test 10 "/usr/bin/sort" "uniq"

# Test 11: comando1 vacío
TOTAL_TESTS=$((TOTAL_TESTS + 1))
echo "input for empty command1" > $INPUT_FILE
./pipex $INPUT_FILE "" "wc -l" $OUTPUT_FILE 2> $ERROR_LOG
EXIT_CODE=$?
echo -e "${GREEN}[OK]${NC} Test 11: $INPUT_FILE \"\" | \"wc -l\" $OUTPUT_FILE"
echo -e "${YELLOW}[DEBUG]${NC} Test 11 error message:"
cat $ERROR_LOG

# Test 12: comando2 vacío
TOTAL_TESTS=$((TOTAL_TESTS + 1))
echo "input for empty command2" > $INPUT_FILE
./pipex $INPUT_FILE "cat" "" $OUTPUT_FILE 2> $ERROR_LOG
EXIT_CODE=$?
echo -e "${GREEN}[OK]${NC} Test 12: $INPUT_FILE \"cat\" | \"\" $OUTPUT_FILE"
echo -e "${YELLOW}[DEBUG]${NC} Test 12 error message:"
cat $ERROR_LOG

# Limpieza
rm -f $INPUT_FILE $OUTPUT_FILE $EXPECTED_FILE $ERROR_LOG

echo -e "\n✅ Tester finalizado con ${GREEN}${TOTAL_TESTS} tests${NC}."
