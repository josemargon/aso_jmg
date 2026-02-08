# 1. Leemos los ficheros
$contenidoRRHH = Get-Content "hr_status_raw.txt"
$contenidoVPN = Get-Content "vpn_logs_raw.txt"

$listaEmpleados = @()

foreach ($linea in $contenidoRRHH) {
    # Si la línea está vacía, pasamos a la siguiente
    if ($linea -eq "") { continue }

    $partes = $linea.Split("|")
    
    $nombreCompleto = $partes[1].Trim()
    $estado = $partes[2].Trim()

    # Sacar usuario: Primera letra nombre + apellido
    # Ejemplo: "JUAN DOE" -> separamos por espacio
    $palabras = $nombreCompleto.Split(" ") 
    
    # Primera letra del nombre (0,1) + Apellido completo
    $usuarioGenerado = $palabras[0].Substring(0,1) + $palabras[1]
    
    # Guardamos en un objeto simple y lo añadimos a la lista
    $empleado = [PSCustomObject]@{
        Usuario = $usuarioGenerado.ToLower()
        Estado  = $estado
    }
    $listaEmpleados += $empleado
}

$reporteFinal = @()

foreach ($linea in $contenidoVPN) {
    # Saltar líneas vacías
    if ($linea -eq "") { continue }

    $partes = $linea.Split("#")
    
    $ip = $partes[0].Trim()
    $fechaSucia = $partes[1].Trim()
    $usuarioSucio = $partes[2].Trim()

    # Si tiene barra invertida (\) cogemos lo de la derecha
    if ($usuarioSucio.Contains("\")) {
        $usuarioLimpio = $usuarioSucio.Split("\")[1]
    } 
    # Si tiene arroba (@) cogemos lo de la izquierda
    elseif ($usuarioSucio.Contains("@")) {
        $usuarioLimpio = $usuarioSucio.Split("@")[0]
    } 
    else {
        $usuarioLimpio = $usuarioSucio
    }

    # Quitar puntos y pasar a minúsculas
    $usuarioLimpio = $usuarioLimpio.Replace(".", "").ToLower()

    # Saltar si es invitado
    if ($usuarioLimpio -eq "invitado_externo") { continue }

    # Reemplazamos todos los símbolos raros por guiones
    $fechaArreglada = $fechaSucia.Replace("_", " ").Replace("/", "-").Replace(".", "-")
    $fechaObjeto = Get-Date $fechaArreglada
    $fechaISO = $fechaObjeto.ToString("yyyy-MM-dd HH:mm")

    
    $empleadoEncontrado = $null
    
    # Recorremos la lista de RRHH que creamos antes para ver si está
    foreach ($emp in $listaEmpleados) {
        if ($emp.Usuario -eq $usuarioLimpio) {
            $empleadoEncontrado = $emp
            break # Ya lo encontramos, dejamos de buscar
        }
    }

    # Decidir si es Incidente
    $incidente = $null
    $estadoRRHH = "DESCONOCIDO"

    if ($empleadoEncontrado -eq $null) {
        # No estaba en la lista
        $incidente = "USUARIO_DESCONOCIDO"
    } else {
        # Sí estaba, miramos su estado
        $estadoRRHH = $empleadoEncontrado.Estado
        
        if ($estadoRRHH -eq "BAJA") {
            $incidente = "ACCESO_NO_AUTORIZADO"
        }
        elseif ($estadoRRHH -eq "SUSPENDIDO") {
            $incidente = "ACCESO_NO_AUTORIZADO"
        }
    }

    # Si hubo incidente, lo guardamos para el CSV
    if ($incidente -ne $null) {
        $objetoSalida = [PSCustomObject]@{
            "FechaHora (ISO)" = $fechaISO
            Usuario           = $usuarioLimpio
            IP_Origen         = $ip
            Estado_RRHH       = $estadoRRHH
            Tipo_Incidente    = $incidente
        }
        $reporteFinal += $objetoSalida
    }
}

$reporteFinal | Format-Table -AutoSize
$reporteFinal | Export-Csv "Reporte_Incidentes.csv" -NoTypeInformation -Encoding UTF8