# haska_tesis_2017

#################
# Script para Praat
# Función: Obtiene valores:
#  1. Duración de
#   a) oclusión
#   b) fricción
#   c) porcentaje de fricción respecto de oclusión
#  2. Cruces por cero (en 20 ms) de la fricativa
#  3. Centro de gravedad de la fricción
# Deja una tabla con todos los datos.
##################
##################


#### Elegir carpeta de trabajo
### cambiar barra / (mac) a \ (otro)

directorioOriginal$ = chooseDirectory$: "Elige la carpeta"
extensionTextGrid$ = ".TextGrid"
extensionSonido$ = ".wav"
especificacionSonido$ = directorioOriginal$ + "/*" + extensionSonido$


# Hacer la lista de todos los objetos Sound de esa carpeta

listaSonido = Create Strings as file list: "listaSonido", especificacionSonido$
totalSonido = Get number of strings
tabla = Create Table with column names: "data", totalSonido, "clase Ninfo sexo palab repet durFon durOcl durFri por100Fricc X0
... difIn0_1 difIn0_2 CdeG"

writeInfoLine: "Atenta, Chrissssss"
for iSonido to totalSonido
	selectObject: listaSonido
	nombreSonido$ = Get string: iSonido
	nombreTextGrid$ = nombreSonido$ - extensionSonido$ + extensionTextGrid$
	sonidoEditando = Read from file: directorioOriginal$ + "/" + nombreSonido$
	Subtract mean
	etiquetaEditando = Read from file: directorioOriginal$ + "/" + nombreTextGrid$
	duracion_audio = Get total duration

#
### Para datos a partir del nombre
#

# 1. Informante
	largo = length(nombreSonido$)
	informante$ = right$(nombreSonido$,5)
	informante$ = left$(informante$,1)

# 2. clase
	clase$ = right$(nombreSonido$,6)
	clase$ = left$(clase$,1)

# 3. sexo
	sexo$ = right$(nombreSonido$,7)
	sexo$ = left$(sexo$,1)

# 4. Repetición

	paraNumeroRepeticion = largo-8

	caracterFinal$ =mid$ (nombreSonido$,paraNumeroRepeticion,1)
	caracterFinalNumero = number (caracterFinal$)

		if caracterFinalNumero = undefined
			caracterFinalNumero = 1
		endif

# 5. Palabra

	if 	caracterFinal$ = "1" or caracterFinal$ = "2" or caracterFinal$ = "3" or caracterFinal$ = "4"
		palabra$ = left$(nombreSonido$,largo-9)
		else
			palabra$ = left$(nombreSonido$,largo-8)
	endif

# Selecciona el TextGrid para calcular duración

select etiquetaEditando

# verifica numero de tiers, intervalos y puntos por tier
nDeTiers = Get number of tiers
nPuntosTier1 = Get number of points: 1
nIntervalosTier2 = Get number of intervals: 2
nIntervalosTier3 = Get number of intervals: 3

if nDeTiers <> 3 or nPuntosTier1 <> 3 or nIntervalosTier2 <> 4 or nIntervalosTier3 <> 3
	appendInfoLine: "Algo anda mal, Chrissss"
	appendInfoLine: "Compara los números entre paréntesis con los números posparéntesis"
	appendInfoLine: "En el TextGrid de ", nombreSonido$, " ocurre..."
	appendInfoLine: "Tiers (3)", nDeTiers
	appendInfoLine: "Puntos en tier 1 (3)", nPuntosTier1
	appendInfoLine: "Intervalos tier 2 (4)", nIntervalosTier2
	appendInfoLine: "Intervalos tier 3 (3)", nIntervalosTier3
endif

# puntos de valores en el tier 1

tiempoMarca1_01= Get time of point: 1, 1
etiquetaMarca1_01$ = Get label of point: 1, 1
tiempoMarca2_01= Get time of point: 1, 2
etiquetaMarca2_01$ = Get label of point: 1, 2
tiempoMarca3_01= Get time of point: 1, 3
etiquetaMarca3_01$ = Get label of point: 1, 3

# etiquetas en el tier 2.

etiquetaOclusion$ =Get label of interval: 2, 2
etiquetaFriccion$ =Get label of interval: 2, 3

if etiquetaOclusion$ ="O" or etiquetaOclusion$ ="o"
	inicioOclusion = Get start time of interval: 2, 2
	finalOclusion = Get end time of interval: 2, 2
	duracionOclusion = finalOclusion-inicioOclusion
elsif duracionOclusion = 0
endif

if etiquetaFriccion$ ="F" or etiquetaFriccion$ ="f"
	inicioFriccion = Get start time of interval: 2, 3
	finalFriccion = Get end time of interval: 2, 3
	duracionFriccion = finalFriccion-inicioFriccion
endif

#porcentaje de oclusión y fricción

duracionTotal = duracionOclusion + duracionFriccion

porcentajeFriccion = duracionFriccion*100/(duracionOclusion + duracionFriccion)

if sexo$ = "H"
rango =5000
else
rango = 5500
endif

select sonidoEditando
spectrog = To Spectrogram: 0.03, rango, 0.002, 20, "Gaussian"
slice = To Spectrum (slice): tiempoMarca2_01
cDeGrav = Get centre of gravity: 2

select spectrog
plus slice
Remove

#CrucesXCero

select sonidoEditando
paraXporCero = To PointProcess (zeroes): 1, "yes", "yes"

inicioCuentaCrucesXCero = tiempoMarca2_01 - 0.015
finalCuentaCrucesXCero = tiempoMarca2_01 + 0.015
desdeXCero = Get nearest index: inicioCuentaCrucesXCero
hastaXCero = Get nearest index: finalCuentaCrucesXCero
crucesXCero = hastaXCero-desdeXCero

select sonidoEditando

paraIntensidad = To Intensity: 100, 0, "yes"
valorIntensidad_1 = Get value at time: tiempoMarca1_01, "Cubic"
valorIntensidad_0 = Get value at time: tiempoMarca2_01, "Cubic"
valorIntensidad_2 = Get value at time: tiempoMarca3_01, "Cubic"
dif_intens_0_1 = valorIntensidad_0 - valorIntensidad_1
dif_intens_0_2 = valorIntensidad_0 - valorIntensidad_2

select tabla
Set string value: iSonido, "clase", clase$
Set string value: iSonido, "Ninfo", informante$
Set string value: iSonido, "sexo", sexo$
Set string value: iSonido, "palab", palabra$
Set numeric value: iSonido, "repet", caracterFinalNumero
Set numeric value: iSonido, "durFon", duracionTotal
Set numeric value: iSonido, "durOcl", duracionOclusion
Set numeric value: iSonido, "durFri", duracionFriccion
Set numeric value: iSonido, "por100Fricc", porcentajeFriccion
Set numeric value: iSonido, "X0", crucesXCero
Set numeric value: iSonido, "difIn0_1", dif_intens_0_1
Set numeric value: iSonido, "difIn0_2", dif_intens_0_2
Set numeric value: iSonido, "CdeG", cDeGrav

select sonidoEditando
plus etiquetaEditando
plus paraXporCero
plus paraIntensidad
Remove

endfor

select listaSonido
Remove
select tabla
appendInfoLine: "Guarde la tabla, Christina"
