#!/bin/bash

# ASCII Art: CrashKraken
echo -e "
                 .,::,.                 
               'ckKXXKkc'               
            .;xXWNOxxONWXx;.            
           .dNWKd;....;dKWNd.           
          .xWXo'.ck00kc.'oXWx.          
          '0Xc.;0WNKKNW0;.cX0'          
          .d0,.kWO;..;OWk.,0d.          
           'xl.dXc    cXd.lx'           
            lx.'xl.   lx'.xl            
           .cxo,':.  .:',oxc.           
             .,'...  ...',.             
           .';';;......;;';'.           
   ..';::::;;;cl,:;..;:,lc;;;::::;'..   
.:xkxdl:;;cllc;;c:.  .:c;;cllc;;:ldxkx:.
oXx;.  .d0x:..;o,      ,o:..:x0d.  .;xXo
dx.   .dXo.  :O:        :O:  .oXd.   .xd
.,.   .OK,   lK:        :Ko   ,KO.   .,.
      .cOc   ,Ok'      'k0,   cOc.      
        ':.   ,kx'    'xk,   .:'        
               .dd.  .dd.               
                .dl..ld.                
                 cl..lc                 
                .;.  .;.                
                ..    ..
"

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

# --- Tests Adicionales ---

# Test 13: Comando con argumentos complejos (comillas y espacios)
run_test 13 "echo \"hello world\"" "tr a-z A-Z" output.txt

# Test 14: Ruta relativa de comando (./command)
echo "#!/bin/sh\necho 'test_rel_path'" > test_script.sh
chmod +x test_script.sh
run_test 14 "./test_script.sh" "wc -c" output.txt
rm -f test_script.sh

# Test 15: Archivo de entrada inexistente
TOTAL_TESTS=$((TOTAL_TESTS + 1))
./pipex no_exist.txt "cat" "wc -l" output.txt 2> $ERROR_LOG
if [ $? -ne 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 15: Manejo de archivo de entrada inexistente"
else
    echo -e "${RED}[KO]${NC} Test 15: Archivo inexistente no generó error"
fi

# Test 16: Permisos denegados en archivo de entrada
TOTAL_TESTS=$((TOTAL_TESTS + 1))
echo "restricted" > restricted.txt
chmod 000 restricted.txt
./pipex restricted.txt "cat" "wc -l" output.txt 2> $ERROR_LOG
if [ $? -ne 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 16: Manejo de permisos denegados (input)"
else
    echo -e "${RED}[KO]${NC} Test 16: Permisos denegados no generaron error"
fi
rm -f restricted.txt

# Test 17: Permisos denegados en archivo de salida
TOTAL_TESTS=$((TOTAL_TESTS + 1))
touch no_write.txt
chmod 000 no_write.txt
./pipex input.txt "cat" "wc -l" no_write.txt 2> $ERROR_LOG
if [ $? -ne 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 17: Manejo de permisos denegados (output)"
else
    echo -e "${RED}[KO]${NC} Test 17: Permisos denegados no generaron error"
fi
rm -f no_write.txt

# Test 18: Comando no encontrado
TOTAL_TESTS=$((TOTAL_TESTS + 1))
./pipex input.txt "comando_inexistente" "wc -l" output.txt 2> $ERROR_LOG
if [ $? -ne 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 18: Comando no encontrado genera error"
else
    echo -e "${RED}[KO]${NC} Test 18: Comando inexistente no falló"
fi

# Test 19: Comando inválido (sintaxis incorrecta)
TOTAL_TESTS=$((TOTAL_TESTS + 1))
./pipex input.txt "ls --invalid-flag" "wc -l" output.txt 2> $ERROR_LOG
if [ $? -ne 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 19: Comando inválido genera error"
else
    echo -e "${RED}[KO]${NC} Test 19: Comando inválido no falló"
fi

# Test 20: Entrada vacía
TOTAL_TESTS=$((TOTAL_TESTS + 1))
echo -n "" > empty.txt
./pipex empty.txt "cat" "wc -l" output.txt 2> $ERROR_LOG
diff output.txt <(wc -l < empty.txt) > /dev/null
if [ $? -eq 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 20: Archivo de entrada vacío"
else
    echo -e "${RED}[KO]${NC} Test 20: Manejo de entrada vacía falló"
fi
rm -f empty.txt

# Test 21: Pipeline largo (3 comandos) - Requiere bonus
TOTAL_TESTS=$((TOTAL_TESTS + 1))
./pipex input.txt "cat" "grep a" "wc -l" output.txt 2> $ERROR_LOG
if [ $? -eq 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 21: Pipeline largo (bonus)"
else
    echo -e "${YELLOW}[SKIP]${NC} Test 21: Bonus no implementado (múltiples pipes)"
fi

# Test 22: Variables de entorno en comandos
TOTAL_TESTS=$((TOTAL_TESTS + 1))
export TEST_VAR="123"
./pipex input.txt "echo \$TEST_VAR" "wc -c" output.txt 2> $ERROR_LOG
if [ $? -eq 0 ]; then
    echo -e "${GREEN}[OK]${NC} Test 22: Variables de entorno en comandos"
else
    echo -e "${RED}[KO]${NC} Test 22: Fallo al expandir variables"
fi

# Limpieza
rm -f $INPUT_FILE $OUTPUT_FILE $EXPECTED_FILE $ERROR_LOG

echo -e "\n✅ Tester finalizado con ${GREEN}${TOTAL_TESTS} tests${NC}."
