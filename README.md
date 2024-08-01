<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cotizador</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f0f0;
            margin: 0;
            padding: 0;
        }

        .container {
            max-width: 800px;
            margin: 20px auto;
            background-color: #fff;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        }

        h1 {
            text-align: center;
            color: #333;
            font-size: 24px;
            margin-bottom: 20px;
        }
    
        #productos, #cotizaciones {
            margin-bottom: 20px;
        }

        #total {
            text-align: right;
            margin-bottom: 20px;
        }

        #total-cotizacion {
            font-weight: bold;
            color: #007bff;
        }

        form {
            display: flex;
            flex-wrap: wrap;
            justify-content: space-between;
            align-items: center;
            margin-top: 20px;
        }

        input[type="text"],
        input[type="number"],
        button {
            width: calc(48% - 10px);
            margin: 5px 0;
            padding: 10px;
            border-radius: 5px;
            border: 1px solid #ccc;
            font-size: 16px;
            box-sizing: border-box;
        }

        button {
            width: 100%;
            color: #fff;
            border: none;
            cursor: pointer;
            transition: background-color 0.3s, transform 0.3s;
        }

        button:hover {
            transform: scale(1.05);
        }

        .btn-primary {
            background-color: #007bff;
        }

        .btn-primary:hover {
            background-color: #0056b3;
        }

        .btn-success {
            background-color: #28a745;
        }

        .btn-success:hover {
            background-color: #218838;
        }

        .btn-danger {
            background-color: #dc3545;
        }

        .btn-danger:hover {
            background-color: #c82333;
        }

        .producto-item {
            margin-bottom: 10px;
            padding: 15px;
            border: 1px solid #ccc;
            border-radius: 5px;
            background-color: #fafafa;
        }

        .producto-item p {
            margin: 5px 0;
            color: #555;
        }

        .total-producto {
            margin-top: 10px;
            font-weight: bold;
            color: #007bff;
        }

        .producto-item button {
            margin-top: 10px;
            padding: 5px 10px;
        }

        .cotizacion-item {
            margin-bottom: 10px;
            padding: 15px;
            border: 1px solid #ccc;
            border-radius: 5px;
            background-color: #fafafa;
        }

        .cotizacion-item p {
            margin: 5px 0;
            color: #555;
        }

        .producto-detalle {
            display: none;
            margin-top: 10px;
            padding-left: 20px;
        }

        .producto-detalle p {
            margin: 5px 0;
        }

        #cotizacion-form {
            display: none;
        }

        @media (max-width: 600px) {
            input[type="text"],
            input[type="number"],
            button {
                width: 100%;
                margin: 5px 0;
            }

            .container {
                margin: 10px;
                padding: 10px;
            }
        }
    </style>
</head>
<body>
    <div id="cotizador" class="container">
        <h1>Cotizador</h1>
        <input type="text" id="nombre-proyecto" placeholder="Nombre del proyecto" style="width: 100%; padding: 10px; margin-bottom: 20px;">
        <div id="productos">
            <!-- Aquí se agregarán dinámicamente los productos -->
        </div>
        <div id="total">
            <p>Total de la cotización: <span id="total-cotizacion">$0.00</span></p>
        </div>
        <form id="formulario">
            <input type="hidden" id="indice-producto">
            <input type="text" id="producto" placeholder="Nombre del producto">
            <input type="number" id="cantidad" placeholder="Cantidad">
            <input type="number" id="precio" placeholder="Precio unitario ($)">
            <button type="button" class="btn-success" onclick="agregarOActualizarProducto()">Agregar/Actualizar Producto</button>
            <button type="button" class="btn-danger" onclick="limpiarFormulario()">Limpiar</button>
            <button type="button" class="btn-primary" onclick="guardarCotizacion()">Guardar Cotización</button>
        </form>
        <button type="button" class="btn-primary" onclick="mostrarCotizaciones()">Ver Cotizaciones Guardadas</button>
    </div>

    <div id="cotizacion-form" class="container">
        <h1>Cotizaciones Guardadas</h1>
        <div id="cotizaciones">
            <!-- Aquí se agregarán dinámicamente las cotizaciones -->
        </div>
        <button type="button" class="btn-primary" onclick="volverAlCotizador()">Volver al Cotizador</button>
    </div>

    <script>
        let productos = [];
        let cotizaciones = JSON.parse(localStorage.getItem('cotizaciones')) || [];
        let indiceEdicion = null;

        function agregarOActualizarProducto() {
            let indiceProducto = document.getElementById('indice-producto').value;
            let producto = document.getElementById('producto').value.trim();
            let cantidad = parseInt(document.getElementById('cantidad').value.trim());
            let precio = parseFloat(document.getElementById('precio').value.trim());

            if (producto !== '' && !isNaN(cantidad) && cantidad > 0 && !isNaN(precio) && precio > 0) {
                let nuevoProducto = {
                    nombre: producto,
                    cantidad: cantidad,
                    precio: precio
                };

                if (indiceProducto === '') {
                    productos.push(nuevoProducto);
                } else {
                    productos[indiceProducto] = nuevoProducto;
                    document.getElementById('indice-producto').value = '';
                }

                mostrarProductos();
                calcularTotal();
                limpiarFormularioInputs(false); // No limpiar el nombre del proyecto
            } else {
                alert('Por favor ingresa un nombre de producto válido, cantidad y precio mayor a cero.');
            }
        }

        function mostrarProductos() {
            let productosDiv = document.getElementById('productos');
            productosDiv.innerHTML = '';

            productos.forEach((producto, index) => {
                let totalProducto = (producto.cantidad * producto.precio).toLocaleString('es-MX', { style: 'currency', currency: 'MXN' });
                let productoHTML = `
                    <div class="producto-item">
                        <p>${producto.nombre}</p>
                        <p>Cantidad: ${producto.cantidad}</p>
                        <p>Precio unitario: $${producto.precio.toFixed(2)}</p>
                        <div class="total-producto">${totalProducto}</div>
                        <button type="button" class="btn-success" onclick="editarProducto(${index})">Editar</button>
                        <button type="button" class="btn-danger" onclick="eliminarProducto(${index})">Eliminar</button>
                    </div>
                `;
                productosDiv.innerHTML += productoHTML;
            });
        }

        function calcularTotal() {
            let total = 0;

            productos.forEach(producto => {
                total += producto.cantidad * producto.precio;
            });

            let totalFormatted = total.toLocaleString('es-MX', { style: 'currency', currency: 'MXN' });
            let totalElement = document.getElementById('total-cotizacion');
            totalElement.textContent = totalFormatted;
        }

        function editarProducto(indice) {
            let producto = productos[indice];
            document.getElementById('producto').value = producto.nombre;
            document.getElementById('cantidad').value = producto.cantidad;
            document.getElementById('precio').value = producto.precio;
            document.getElementById('indice-producto').value = indice;
        }

        function eliminarProducto(indice) {
            productos.splice(indice, 1);
            mostrarProductos();
            calcularTotal();
        }

        function limpiarFormulario() {
            productos = [];
            mostrarProductos();
            calcularTotal();
            limpiarFormularioInputs(true); // Limpiar todo incluyendo el nombre del proyecto
        }

        function limpiarFormularioInputs(limpiarNombreProyecto) {
            document.getElementById('producto').value = '';
            document.getElementById('cantidad').value = '';
            document.getElementById('precio').value = '';
            document.getElementById('indice-producto').value = '';
            if (limpiarNombreProyecto) {
                document.getElementById('nombre-proyecto').value = '';
            }
        }

        function guardarCotizacion() {
            const nombreProyecto = document.getElementById('nombre-proyecto').value.trim();
            const totalCotizacion = document.getElementById('total-cotizacion').textContent.trim();

            if (productos.length === 0) {
                alert('Agrega al menos un producto para guardar la cotización.');
                return;
            }

            if (nombreProyecto === '') {
                alert('Por favor ingresa un nombre de proyecto.');
                return;
            }

            let cotizacion = {
                nombre: nombreProyecto,
                total: totalCotizacion,
                productos: [...productos]
            };

            if (indiceEdicion === null) {
                cotizaciones.push(cotizacion);
            } else {
                cotizaciones[indiceEdicion] = cotizacion;
                indiceEdicion = null;
            }

            localStorage.setItem('cotizaciones', JSON.stringify(cotizaciones));
            limpiarFormulario();
        }

        function mostrarCotizaciones() {
            let cotizacionesDiv = document.getElementById('cotizaciones');
            cotizacionesDiv.innerHTML = '';

            cotizaciones.forEach((cotizacion, index) => {
                let cotizacionHTML = `
                    <div class="cotizacion-item">
                        <p>${cotizacion.nombre} - Total: ${cotizacion.total}</p>
                        <div class="total-cotizacion">${cotizacion.total}</div>
                        <button type="button" class="btn-primary" onclick="toggleDetalleCotizacion(${index})">Ver/Agregar Productos</button>
                        <button type="button" class="btn-success" onclick="editarCotizacion(${index})">Editar</button>
                        <button type="button" class="btn-danger" onclick="eliminarCotizacion(${index})">Eliminar</button>
                        <div class="producto-detalle" id="detalle-cotizacion-${index}">
                            ${cotizacion.productos.map(producto => `
                                <p>${producto.nombre} - Cantidad: ${producto.cantidad} - Precio unitario: $${producto.precio.toFixed(2)} - Total: ${(producto.cantidad * producto.precio).toLocaleString('es-MX', { style: 'currency', currency: 'MXN' })}</p>
                            `).join('')}
                        </div>
                    </div>
                `;
                cotizacionesDiv.innerHTML += cotizacionHTML;
            });

            document.getElementById('cotizador').style.display = 'none';
            document.getElementById('cotizacion-form').style.display = 'block';
        }

        function toggleDetalleCotizacion(indice) {
            let detalleDiv = document.getElementById(`detalle-cotizacion-${indice}`);
            if (detalleDiv.style.display === 'none' || detalleDiv.style.display === '') {
                detalleDiv.style.display = 'block';
            } else {
                detalleDiv.style.display = 'none';
            }
        }

        function editarCotizacion(indice) {
            let cotizacion = cotizaciones[indice];
            document.getElementById('nombre-proyecto').value = cotizacion.nombre;
            productos = [...cotizacion.productos];
            mostrarProductos();
            calcularTotal();
            indiceEdicion = indice;
            volverAlCotizador();
        }

        function eliminarCotizacion(indice) {
            cotizaciones.splice(indice, 1);
            localStorage.setItem('cotizaciones', JSON.stringify(cotizaciones));
            mostrarCotizaciones();
        }

        function volverAlCotizador() {
            document.getElementById('cotizador').style.display = 'block';
            document.getElementById('cotizacion-form').style.display = 'none';
        }
    </script>
</body>
</html>
