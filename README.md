# tecnicasdesimulacion
sistema de linea de espera
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class lineaespera extends Controller
{
    public function index()
    {
    	return view('inicio.lineaespera');
    }

    public function create()
    {
    	# code...
    }

    public function show($id)
    {
    	# code...
    }

    public function store(Request $request)
    {
        // Creacion de un arreglo de numeros aleatorio para los valores de llegada
        $aleatorios_llegada = [];
        // Se iguala a una variable la cantidad de numeros aleatorios que deseamos
        $eventos = $request->eventos;
        // se realiza un bucle dependiendo de la cantidad de numeros que deseemos
        for ($i=0; $i < $eventos; $i++) { 
            // Se crea la variable que contendra el numero aleatorio por medio de la funcion php rand, este numero tendra parte entera
            $aleatorio = 1000/rand(1,1000);
            // Se iguala otra variable para quitarle la parte entera al aleatorio y redondearlo a tres decimales
            $randomx = round(($aleatorio - intval($aleatorio)),3);
            // se agrega al arreglo el numero aleatorio generado
            array_push($aleatorios_llegada,$randomx);
        }
        /********Se realiza el mismo proceso para los aleatorios de valores de servicio*/
        $aleatorios_servicio = [];
        for ($i=0; $i < $eventos; $i++) { 
            $aleatorio = 1000/rand(1,1000);
            $randomx = round(($aleatorio - intval($aleatorio)),3);
            array_push($aleatorios_servicio,$randomx);
        }

        $bandera = 0;
        if (isset($request->tipo)) {
            $bandera = $request->tipo;
        }
        // finalmente se retorna la tabla con valores aleatorios de llegada y de servicio
        return view('desnudas.aleatorios_llegada_servicio',compact('aleatorios_llegada','aleatorios_servicio','bandera'));
    }

    public function update(Request $request)
    {
    	# code...
    }

    public function destroy($request)
    {
    	# code...
    }

    public function calcular_linea_espera(Request $request)
    {
        $llegadas = $request->aleatorio_llegada;
        $servicios = $request->aleatorio_servicio;
        $contador = 0;
        $coun = 0;
        $lamda = $request->lamda;
        $niu = $request->niu;
        $datos = [];

        $fila[0] = $contador;
        $fila[1] = ' - ';
        $fila[2] = ' - ';
        $fila[3] = ' - ';
        $fila[4] = ' - ';
        $fila[5] = 0;
        $fila[6] = 0;
        $fila[7] = 0;
        $fila[8] = 0;
        $fila[9] = 0;
        array_push($datos,$fila);
        for ($i=0; $i < count($llegadas); $i++) {
            $contador = $contador + 1;
            $fila[0] = $contador;
            $llegada = $llegadas[$i];
            $fila[1] = $llegada;
            $servicio = $servicios[$i];
            $fila[2] = $servicio;
            $entre_llegadas = (-1/$lamda)*log($llegada);
            $fila[3] = round($entre_llegadas,2);
            $t_servicio = (-1/$niu)*log($servicios[$i]);
            $fila[4] = round($t_servicio,2);
            $llegada_exacta = round($datos[$i][5] + $entre_llegadas,2);
            $fila[5] = $llegada_exacta;
            $inicio_servicio = max($llegada_exacta,$datos[$i][7]);
            $fila[6] = round($inicio_servicio,2);
            $fin_servicio = round($inicio_servicio + $t_servicio,2);
            $fila[7] = $fin_servicio;
            $t_espera = round($inicio_servicio - $llegada_exacta,2);
            $fila[8] = $t_espera;
            $t_sistema = round($t_espera + $t_servicio,2);
            $fila[9] = $t_sistema;
            array_push($datos, $fila);
        }

        return view('desnudas.lineaesperatabla',compact('datos','contador'));
    }

    public function montecarlo()
    {
        return view('inicio.lineaesperamontecarlo');
    }

    public function generar_tab_pro(Request $request)
    {
        // recibe la cantidad de campos tanto valores como probabilidades
        $valor_n = $request->cantidad;
        // el tipo de eventos que seran, puede ser 1 que corresponde a eventos de llegada o puede ser 2 que corresponde a eventos de servicio
        $tipo = $request->tipo;
        // esto retorna la vista en donde se generan las tablas 
        return view('desnudas.tabla_tab_pro',compact('valor_n','tipo'));
    }

    public function calcularlineamontecarlo(Request $request)
    {
        // Crear tabla de probabilidad de llegada
        $tabla_pro_llegada = [];
        // crear una variable acumuladora que ira acumulando los valores de la probabilidad acumulada
        $acumulador_pro_lle = 0;
        // esta variable lo que hace es Acumular los rangos menores de la tabla
        $acumulador_rango_menor = 0;
        // se genera un bucle dependiendo del numero de probabilidades y valores que contendran la tabla de llegada
        for ($i=0; $i < count($request->valor1); $i++) { 
            // en el arreglo multidimensional se guarda en la posicion i0 el valor de la llegada
            $tabla_pro_llegada[$i][0] = $request->valor1[$i];
            // en el arreglo multidimensional se guarda en la posicion i1 el valor de la probabilidad
            $tabla_pro_llegada[$i][1] = $request->probabilidad1[$i];
            // se hace uso de la variable acumuladora para la probabilidad acumulada
            $acumulador_pro_lle = $acumulador_pro_lle + $request->probabilidad1[$i];
            // en la posicion i2 se guarda la probabilidad acumulada
            $tabla_pro_llegada[$i][2] = $acumulador_pro_lle;
            // en la posicion i3 se guarda el rango menor y se redondea a tres decimales por medio de la funcion round
            $tabla_pro_llegada[$i][3] = round($acumulador_rango_menor,3);
            // se actualiza el acumulador aumentandole 0.001 para el rango
            $acumulador_rango_menor = round($acumulador_pro_lle,3) + 0.001;
            // en la posicion i4 del arreglo se guarda el rango mayor
            $tabla_pro_llegada[$i][4] = $acumulador_pro_lle;
        }

        /*Este proceso es el mismo que el anterior, la diferencia es que se genera para los tiempos de servicios*/
        $tabla_pro_servicio = [];
        // Crear tabla de probabilidad de Servicios
        $acumulador_pro_ser = 0;
        $acumulador_rango_menor_ser = 0;
        for ($i=0; $i < count($request->valor2); $i++) { 
            $tabla_pro_servicio[$i][0] = $request->valor2[$i];
            $tabla_pro_servicio[$i][1] = $request->probabilidad2[$i];
            $acumulador_pro_ser = $acumulador_pro_ser + $request->probabilidad2[$i];
            $tabla_pro_servicio[$i][2] = $acumulador_pro_ser;
            $tabla_pro_servicio[$i][3] = round($acumulador_rango_menor_ser,3);
            $acumulador_rango_menor_ser = round($acumulador_pro_ser,3) + 0.001;
            $tabla_pro_servicio[$i][4] = $acumulador_pro_ser;
        }

        // Se crea un arreglo que sera multidimensional para almacenar los resultados de la simulacion
        $tabla_calculada = [];
        // Esta variabe sera util para identificar el numero de fila
        $contador_res = 1;
        // esta es la hora de llegada exacta que se inicializa con cero
        $hora_llegada_exacta = 0;
        // al igual que la llegada exacta el fin de servicio tambien se inicializa con cero
        $hora_fin_servicio = 0;
        // Se realiza el bucle la cantidad de veces de numeros aleatorios enviados
        for ($i=0; $i < count($request->aleatorio_llegada); $i++) { 
            // en la posicion i0 del arreglo se guarda el numero de fila
            $tabla_calculada[$i][0] = $contador_res;
            // en la posicion i1 del arreglo se guarda el aleatorio de llegada
            $tabla_calculada[$i][1] = $request->aleatorio_llegada[$i];
            // en la posicion i2 del arreglo se guarda el aleatorio de servicio
            $tabla_calculada[$i][2] = $request->aleatorio_servicio[$i];
            $t_entre_llegadas = buscar_en_pro($tabla_pro_llegada, $request->aleatorio_llegada[$i]);
            $tabla_calculada[$i][3] = $t_entre_llegadas;
            $t_servicio = buscar_en_pro($tabla_pro_servicio, $request->aleatorio_servicio[$i]);
            $tabla_calculada[$i][4] = $t_servicio;
            $tabla_calculada[$i][5] = $hora_llegada_exacta + $tabla_calculada[$i][3];
            $hora_llegada_exacta = $hora_llegada_exacta + $tabla_calculada[$i][3];
            $tabla_calculada[$i][6] = maximo($tabla_calculada[$i][5],$hora_fin_servicio);
            $hora_fin_servicio = $tabla_calculada[$i][6] + $tabla_calculada[$i][4];
            $tabla_calculada[$i][7] = $hora_fin_servicio;
            $tabla_calculada[$i][8] = $tabla_calculada[$i][6] - $tabla_calculada[$i][5];
            $tabla_calculada[$i][9] = $tabla_calculada[$i][8] + $tabla_calculada[$i][4];
            $contador_res = $contador_res + 1;
        }

        return view('desnudas.lineaesperamontecalo',compact('tabla_pro_llegada','tabla_pro_servicio','tabla_calculada'));
    }
}
