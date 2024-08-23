# CalculadoraBison

# Ejecución de calculadora con Bison
**Nombres:Miguel Fernández, Kevin García, Camilo López, Juan Márquez**
## Introducción
## 1) Instalación de Bison
Para realizar la instalación de Bison, simplemente utilizamos el comando `sudo apt install bison` desde una terminal para que este se descargue en el sistema. Si la instalación es correcta, al ejecutar el comando `bison --version`, podremos ver la versión instalada de Bison.

![Instalación desde terminal](instalacion.png)

## 2) Ejecución de la calculadora
Para la ejecución de la calculadora, primero debemos compilarla con los archivos disponibles en la carpeta `prueba`.

![Carpeta](carpeta.png)

Como se puede ver, son un total de 5 archivos que contienen el código para la calculadora; sin embargo, vamos a crear un ejecutable al compilar los archivos `fb1-5.tab.c` y `lex.yy.c` usando, desde una terminal, el comando `gcc -o calculadora fb1-5.tab.c lex.yy.c -lfl`.

![Compilación](compilacion.png)

Como podemos ver, al momento de compilar, nos da varias advertencias de errores que encuentra al compilar; pero, aun así, nos genera un archivo ejecutable, el cual se puede ejecutar desde una terminal con la carpeta abierta, usando el comando `./calculadora`.

![Ejecutable](ejecutable.png)

### 2.1) Prueba de entrada
Después de verificar que la ejecución del programa es correcta, procederemos a probar el programa con varias operaciones matemáticas.

![Probando](prueba.png)

### 2.2) Manejo de errores
En este caso, estaremos probando entradas inválidas para el programa, tales como el ingreso de caracteres, la división por 0 y las operaciones incompletas.

Comenzaremos con el ingreso de caracteres entre las operaciones y también las operaciones entre caracteres.

![Caracteres](caracteres.png)

Como podemos ver, el ingreso de caracteres entre operaciones no afecta el resultado; sin embargo, se informa la presencia de caracteres no identificados, y en caso de no haber números, se finaliza el programa debido a un `syntax error`.

Ahora haremos una sencilla prueba al realizar una división por cero y también al intentar ejecutar una operación incompleta.

![División por cero](cero.png)

Como podemos ver, al intentar dividir entre cero, siempre nos da el mismo resultado de error y finalización del programa, sin importar cómo varíe la operación para intentar evitar esto.

![Operación incompleta](incompleta.png)

En este caso, vemos también que el resultado de una operación incompleta siempre es el mismo; pero también está el caso donde, a pesar de estar incompleta la operación, la salida es diferente, como en la operación con un cero. En este caso, la operación de división estaba completa, pero había signos de más que dejaban la operación total incompleta, lo que nos puede indicar una jerarquía sobre el manejo de errores por parte de la calculadora.

### 2.3) Verificación de salida
La calculadora da una salida correcta a las operaciones que se le ingresan, así incluyan paréntesis o no. Incluso se usó una operación con un resultado ambiguo, y dio el resultado que la mayoría de calculadoras de equipos electrónicos dan, lo que nos permite decir que los cálculos realizados por el programa son correctos.

## 3) Explicación 
### 3.1) Tokenización
El lexer (fb1-5.l) define las reglas de tokenización, esto es, cómo convertir las partes de la entrada en tokens. En este caso, tenemos las siguientes declaraciones de reglas

```
%%
"+"	{ return ADD; }
"-"	{ return SUB; }
"*"	{ return MUL; }
"/"	{ return DIV; }
"|"     { return ABS; }
"("     { return OP; }
")"     { return CP; }
[0-9]+	{ yylval = atoi(yytext); return NUMBER; }

\n      { return EOL; }
"//".*  
[ \t]   { /* ignore white space */ }
.	{ yyerror("Mystery character %c\n", *yytext); }
%%
```

1. Se definen los operadores aritméticos `+, -, *, /`, los cuales se reconocen directamente y se devuelve un token específico para cada uno: `ADD, SUB, MUL, DIV` respectivamente.
2. Se define el operador valor absoluto `|`, el cual devuelve el token `ABS`.
3. Se definen los paréntesis `(` y `)` los cuales devuelven los tokens `OP (Open Parenthesis)` y `CP (Close Parenthesis)`, respectivamente. Estos son esenciales para manejar la jerarquía y agrupación de operaciones en una expresión dada.
4. Usando `[0-9]+` se define una regla para reconocer una sequencia de uno o más dígitos. Adicionalmente, `yylval = atoi(yytext)` convierte la secuencia de caracteres obtenida, a un número entero, y `return NUMBER` retorna el token `NUMBER` al parser, indicando que se ha encontrado un número.
5. La regla `[ \t]` ignora los espacios en blanco y las tabulaciones.
6. La regla `\n` maneja los saltos de línea.
7. La regla `.` se activa para cualquier otro carácter que no coincida con las reglas anteriores, permitiendo que el parser lo maneje. `{ yyerror("Mystery character %c\n", *yytext); }` hace que cualquier carácter que no coincida con las reglas anteriores será considerado un "carácter misterioso". Esto activa un error (yyerror) y ayuda a identificar entradas no válidas o errores tipográficos

De manera conjunta, cuando la calculadora recibe una expresión como `3 + 4 * 2`, el lexer identifica 3 como un número, lo convierte a un entero y devuelve un token `NUMBER`. Luego, identifica `+` como un operador y devuelve el carácter `+` como un token.
Esto continúa hasta que toda la expresión ha sido tokenizada. Cada token se envía al parser, que construye un árbol de sintaxis para la evaluación final de la expresión.

### 3.2) Análisis

El parser (`fb1-5.y`) toma los tokens generados por el lexer y los organiza en un árbol de sintaxis según las reglas definidas en la gramática.
Este árbol de sintaxis refleja la estructura lógica de la expresión matemática, respetando la precedencia de operadores y el orden de las operaciones.
En primer lugar, tenemos las definiciones de tokens

```
%token NUMBER
%token ADD SUB MUL DIV ABS
%token OP CP
%token EOL
```



### 3.3) Evaluación
#### Proceso de Evaluación

El árbol de sintaxis se recorre desde las hojas hacia la raíz.
Cada regla de producción en la gramática tiene un bloque de código asociado que realiza la operación correspondiente:

  - Operaciones aritméticas: Por ejemplo, en exp `ADD exp { $$ = $1 + $3; }`, los valores de las subexpresiones a la izquierda y derecha     `($1   y $3)`se suman.

  - Prioridad de operadores: Las expresiones se evalúan según la prioridad definida en la gramática. Por ejemplo, la multiplicación y la     división tienen una prioridad más alta que la suma y la resta.

Paréntesis y operadores unarios: Los paréntesis permiten alterar la precedencia de los operadores, y el operador de valor absoluto `(|)` se maneja en las reglas correspondientes.

4. Producción del Resultado Final:

Después de que la expresión completa ha sido evaluada, el resultado se almacena en la variable especial $$, que se propaga hacia arriba en el árbol de sintaxis.
Finalmente, el resultado de la expresión completa se imprime `(printf("= %d\n> ", $2);)`.

Ejemplo de Evaluación
Para una entrada como `3 + 5 * 2`, el proceso es el siguiente:

Tokens Generados:

- `´3´ (NUMBER)`
- `´+´ (ADD)`
- `´5´ (NUMBER)`
- `´*´ (MUL)`
- `´2´ (NUMBER)`
  
Construcción del Árbol de Sintaxis:

**Construcción del Árbol de Sintaxis**:
- **Árbol**:
  ```plaintext
      +
     / \
    3   *
       / \
      5   2


    - **Evaluación**:
      - Primero se evalúa `5 * 2` que da `10`.
      - Luego, se evalúa `3 + 10` que da `13`.

Según la gramática, `5 * 2` se evalúa primero porque la multiplicación tiene mayor prioridad.
Luego, `3 + (5 * 2)` se evalúa.
Resultado Final:

El resultado es `13`, que se imprime como salida.
Este flujo desde la entrada hasta la producción del resultado final es cómo se evalúa la expresión utilizando un árbol de sintaxis en esta calculadora simple.


