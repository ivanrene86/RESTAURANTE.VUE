<template>
  <div class="app-container">
    <!-- Header con perfil y despachos -->
    <header class="header">
      <div class="header-left">
        <h1>🍽️ Restaurant</h1>
        <div class="despachos">
          <span>📦 Despachos hoy: </span>
          <strong>{{ despachosHoy }}</strong>
        </div>
      </div>
      <div class="header-right">
        <button class="btn-header" @click="abrirModalProducto()">➕ Nuevo</button>
        <button class="btn-header btn-factura-header" @click="mostrarCarrito = true" :disabled="carrito.length === 0">📄 Factura</button>
        <div class="perfil" v-if="mostrarPerfil" @click="mostrarPerfil = false">
          <div class="perfil-info">
            <span class="perfil-nombre">{{ mesero.nombre }}</span>
            <span class="perfil-id">ID: {{ mesero.id }}</span>
          </div>
          <img :src="mesero.foto" alt="Foto mesero" class="perfil-foto" />
        </div>
        <button v-else class="btn-perfil" @click="mostrarPerfil = true">👤 Ver Perfil</button>
      </div>
    </header>

    <!-- Vista de Perfil del Mesero -->
    <div v-if="!mostrarPerfil" class="perfil-view">
      <div class="perfil-card">
        <h2>👤 Perfil del Mesero</h2>
        <img :src="mesero.foto" alt="Foto" class="perfil-grande" />
        <p><strong>Nombre:</strong> {{ mesero.nombre }}</p>
        <p><strong>ID:</strong> {{ mesero.id }}</p>
        <p><strong>Turno:</strong> {{ mesero.turno }}</p>
        <p><strong>Mesas atendidas:</strong> {{ mesero.mesasAtendidas }}</p>
        <button class="btn-volver" @click="mostrarPerfil = true">← Volver</button>
      </div>
    </div>

    <!-- Vista Principal: Productos con Carrito Lateral -->
    <div v-else class="main-layout">
      <!-- Grid de Productos -->
      <div class="productos-grid">
        <div v-for="producto in productos" :key="producto.id" class="product-card">
          <button class="btn-eliminar-producto" @click="eliminarProducto(producto.id)">🗑️</button>
          <img :src="producto.imagen" :alt="producto.nombre" class="product-image" />
          <h3>{{ producto.nombre }}</h3>
          <p class="product-desc">{{ producto.descripcion }}</p>
          <p class="product-precio">${{ producto.precio }}</p>
          <p class="product-stock" :class="{ 'stock-bajo': producto.stock <= 3 }">
            Disponibles: {{ producto.stock }}
          </p>
          
          <div class="product-actions">
            <input 
              type="number" 
              v-model.number="producto.cantidadSeleccionada" 
              min="1" 
              :max="producto.stock"
              class="cantidad-input"
            />
            <button 
              class="btn-agregar" 
              @click="agregarAlCarrito(producto)"
              :disabled="producto.stock === 0"
            >
              Agregar al carrito
            </button>
          </div>
        </div>
      </div>

      <!-- Carrito Lateral -->
      <aside class="carrito-lateral" :class="{ 'abierto': mostrarCarrito }">
        <div class="carrito-header">
          <h3>🛒 Carrito de Pedidos</h3>
          <button class="btn-cerrar-carrito" @click="mostrarCarrito = false">✕</button>
        </div>
        
        <div v-if="carrito.length === 0" class="carrito-vacio">
          <p>El carrito está vacío</p>
          <span class="carrito-vacio-icon">🛒</span>
        </div>
        
        <div v-else class="carrito-items">
          <div v-for="(item, index) in carrito" :key="index" class="carrito-item">
            <div class="carrito-item-info">
              <span class="carrito-nombre">{{ item.nombre }}</span>
              <span class="carrito-precio">${{ item.precio }} x {{ item.cantidad }}</span>
              <span class="carrito-subtotal">= ${{ item.precio * item.cantidad }}</span>
            </div>
            <button class="btn-eliminar" @click="eliminarDelCarrito(index)">🗑️</button>
          </div>
        </div>
        
        <div v-if="carrito.length > 0" class="carrito-footer">
          <div class="carrito-total-final">
            <span>Total:</span>
            <strong>${{ obtenerTotalCarrito() }}</strong>
          </div>
          
          <button class="btn-factura" @click="generarFactura">📄 Generar Factura (PDF)</button>
        </div>
      </aside>
    </div>

    <!-- Modal de Factura -->
    <div v-if="mostrarFactura" class="modal-factura" @click.self="mostrarFactura = false">
      <div class="factura-content">
        <h2>📄 CARRITO</h2>
        <div class="factura-header">
          <p><strong>Restaurant</strong></p>
          <p>Fecha: {{ obtenerFechaActual() }}</p>
          <p>Mesero: {{ mesero.nombre }} (ID: {{ mesero.id }})</p>
        </div>
        
        <table class="factura-tabla">
          <thead>
            <tr>
              <th>Producto</th>
              <th>Cant</th>
              <th>Precio</th>
              <th>Subtotal</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in carrito" :key="index">
              <td>{{ item.nombre }}</td>
              <td>{{ item.cantidad }}</td>
              <td>${{ item.precio }}</td>
              <td>${{ item.precio * item.cantidad }}</td>
            </tr>
          </tbody>
        </table>
        
        <div class="factura-total">
          <h3>TOTAL: ${{ obtenerTotalCarrito() }}</h3>
        </div>
        
        <div class="factura-footer">
          <p>¡Gracias por su visita!</p>
          <p class="pdf-mensaje">📥 El PDF se descargó automáticamente</p>
          <button class="btn-cerrar" @click="cerrarFactura">Cerrar</button>
        </div>
      </div>
    </div>

    <!-- Modal para nuevo producto -->
    <div v-if="mostrarModalProducto" class="modal-factura" @click.self="mostrarModalProducto = false">
      <div class="factura-content">
        <h2>➕ Nuevo Producto</h2>
        
        <div class="form-group">
          <label>Nombre:</label>
          <input type="text" v-model="nuevoProducto.nombre" placeholder="Nombre del producto" />
        </div>
        
        <div class="form-group">
          <label>Descripción:</label>
          <input type="text" v-model="nuevoProducto.descripcion" placeholder="Descripción" />
        </div>
        
        <div class="form-group">
          <label>Precio ($):</label>
          <input type="number" v-model.number="nuevoProducto.precio" min="1" />
        </div>
        
        <div class="form-group">
          <label>Stock:</label>
          <input type="number" v-model.number="nuevoProducto.stock" min="1" />
        </div>
        
        <div class="form-group">
          <label>Imagen (URL):</label>
          <input type="text" v-model="nuevoProducto.imagen" placeholder="https://...jpg" />
        </div>
        
        <div v-if="alertaProducto" class="alerta-producto">{{ alertaProducto }}</div>
        <div class="modal-actions">
          <button class="btn-cerrar" @click="mostrarModalProducto = false">Cancelar</button>
          <button class="btn-factura" @click="guardarProducto">💾 Guardar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue';
import jsPDF from 'jspdf';

const mostrarPerfil = ref(true);
const mostrarCarrito = ref(false);
const mostrarFactura = ref(false);
const mostrarModalProducto = ref(false);
const despachosHoy = ref(0);

const nuevoProducto = reactive({
  nombre: '',
  descripcion: '',
  precio: 0,
  stock: 0,
  imagen: ''
});

const alertaProducto = ref('');

const mesero = reactive({
  id: 'MES-001',
  nombre: 'Juan Pérez',
  turno: 'Matutino (6:00 - 14:00)',
  mesasAtendidas: 12,
  foto: 'https://api.dicebear.com/7.x/avataaars/svg?seed=Juan'
});

const productos = ref([
  { id: 1, nombre: 'Bistec en Salsa', descripcion: 'Bistec de res con salsa', precio: 45000, stock: 8, imagen: 'https://images.unsplash.com/photo-1544025162-d76694265947?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 2, nombre: 'Pollo Asado', descripcion: 'Pollo asado con hierbas', precio: 38000, stock: 10, imagen: 'https://images.unsplash.com/photo-1598103442097-8b74394b95c6?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 3, nombre: 'Pasta Carbonara', descripcion: 'Pasta con tocino y crema', precio: 42000, stock: 5, imagen: 'https://images.unsplash.com/photo-1612874742237-6526221588e3?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 4, nombre: 'Enchiladas', descripcion: 'Enchiladas verdes', precio: 35000, stock: 10, imagen: 'https://images.unsplash.com/photo-1565299585323-38d6b0865b47?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 5, nombre: 'Tacos al Pastor', descripcion: '3 tacos con cebolla', precio: 28000, stock: 10, imagen: 'https://images.unsplash.com/photo-1551504734-5ee1c4a1479b?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 6, nombre: 'Hamburguesa', descripcion: 'Hamburguesa con queso', precio: 32000, stock: 7, imagen: 'https://images.unsplash.com/photo-1568901346375-23c9450c58cd?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 7, nombre: 'Pizza Margarita', descripcion: 'Pizza italiana', precio: 48000, stock: 4, imagen: 'https://images.unsplash.com/photo-1574071318508-1cdbab80d002?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 8, nombre: 'Sushi Roll', descripcion: 'Rol de salmón', precio: 55000, stock: 6, imagen: 'https://images.unsplash.com/photo-1579584425555-c3ce17fd4351?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 9, nombre: 'Paella', descripcion: 'Arroz con mariscos', precio: 62000, stock: 3, imagen: 'https://images.unsplash.com/photo-1534080564583-6be75777b70a?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 10, nombre: 'Sandwich', descripcion: 'Sandwich de jamón', precio: 25000, stock: 10, imagen: 'https://images.unsplash.com/photo-1528735602780-2552fd46c7af?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 11, nombre: 'Salteado de Verduras', descripcion: 'Verduras mixtas', precio: 30000, stock: 9, imagen: 'https://images.unsplash.com/photo-1512621776951-a57141f2eefd?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 12, nombre: 'Filete de Pescado', descripcion: 'Pescado fresco', precio: 52000, stock: 5, imagen: 'https://images.unsplash.com/photo-1519708227418-c8fd9a32b7a2?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 13, nombre: 'Churrasco', descripcion: 'Carne a la parrilla', precio: 58000, stock: 4, imagen: 'https://images.unsplash.com/photo-1558030006-450675393462?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 14, nombre: 'Lasagna', descripcion: 'Lasagna de carne', precio: 44000, stock: 7, imagen: 'https://images.unsplash.com/photo-1574894709920-11b28e7367e3?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 15, nombre: 'Burrito', descripcion: 'Burrito con frijoles', precio: 29000, stock: 10, imagen: 'https://images.unsplash.com/photo-1626700051175-6818013e1d4f?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 16, nombre: 'Alitas BBQ', descripcion: 'Alitas con salsa', precio: 36000, stock: 8, imagen: 'https://images.unsplash.com/photo-1527477396000-e27163b481c2?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 17, nombre: 'Risotto', descripcion: 'Risotto con champiñones', precio: 47000, stock: 6, imagen: 'https://images.unsplash.com/photo-1476124369491-e7addf5db371?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 18, nombre: 'Ceviche', descripcion: 'Ceviche de pescado', precio: 41000, stock: 5, imagen: 'https://images.unsplash.com/photo-1535399831218-d5bd36d1a6b3?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 19, nombre: 'Tortilla Española', descripcion: 'Tortilla con papas', precio: 26000, stock: 10, imagen: 'https://www.goya.com/wp-content/uploads/2023/10/tortilla-espan-ola-potato-omelet.jpg', cantidadSeleccionada: 1 },
  { id: 20, nombre: 'Parrillada', descripcion: 'Variedad de carnes', precio: 75000, stock: 2, imagen: 'https://images.unsplash.com/photo-1555939594-58d7cb561ad1?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 21, nombre: 'Sopa de Mariscos', descripcion: 'Sopa con langosta', precio: 50000, stock: 4, imagen: 'https://images.unsplash.com/photo-1547592166-23ac45744acd?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 22, nombre: 'Empanadas', descripcion: '3 empanadas', precio: 22000, stock: 10, imagen: 'https://images.unsplash.com/photo-1603360946369-dc9bb6258143?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 23, nombre: 'Cocteles', descripcion: 'Cocteles de frutas', precio: 18000, stock: 10, imagen: 'https://images.unsplash.com/photo-1553530666-ba11a7da3888?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 24, nombre: 'Flan', descripcion: 'Flan de caramelo', precio: 15000, stock: 10, imagen: 'https://images.unsplash.com/photo-1470124182917-cc6e71b22ecc?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 25, nombre: 'Cheesecake', descripcion: 'Pastel de queso', precio: 19000, stock: 8, imagen: 'https://images.unsplash.com/photo-1533134242443-d4fd215305ad?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 26, nombre: 'Brownie', descripcion: 'Brownie con nueces', precio: 16000, stock: 10, imagen: 'https://images.unsplash.com/photo-1606313564200-e75d5e30476c?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 27, nombre: 'Café Latte', descripcion: 'Café con leche', precio: 12000, stock: 10, imagen: 'https://images.unsplash.com/photo-1572442388796-11668a67e53d?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 28, nombre: 'Jugo Natural', descripcion: 'Jugo de naranja', precio: 8000, stock: 10, imagen: 'https://images.unsplash.com/photo-1600271886742-f049cd451bba?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 29, nombre: 'Agua Mineral', descripcion: 'Agua sin gas', precio: 6000, stock: 10, imagen: 'https://images.unsplash.com/photo-1523362628745-0c100150b504?w=300&h=200&fit=crop', cantidadSeleccionada: 1 },
  { id: 30, nombre: 'Refresco', descripcion: 'Refresco de cola', precio: 7000, stock: 10, imagen: 'https://images.unsplash.com/photo-1581006852262-e4307cf6283a?w=300&h=200&fit=crop', cantidadSeleccionada: 1 }
]);

const carrito = ref([]);

const agregarAlCarrito = (producto) => {
  if (producto.stock > 0 && producto.cantidadSeleccionada > 0) {
    const cantidad = Math.min(producto.cantidadSeleccionada, producto.stock);
    
    const existente = carrito.value.find(item => item.id === producto.id);
    
    if (existente) {
      existente.cantidad += cantidad;
    } else {
      carrito.value.push({
        id: producto.id,
        nombre: producto.nombre,
        precio: producto.precio,
        cantidad: cantidad
      });
    }
    
    producto.stock -= cantidad;
    producto.cantidadSeleccionada = 1;
  }
};

const eliminarDelCarrito = (index) => {
  const item = carrito.value[index];
  const producto = productos.value.find(p => p.id === item.id);
  
  if (producto) {
    producto.stock += item.cantidad;
  }
  
  carrito.value.splice(index, 1);
};

const obtenerTotalCarrito = () => {
  let total = 0;
  for (let i = 0; i < carrito.value.length; i++) {
    total += carrito.value[i].precio * carrito.value[i].cantidad;
  }
  return total;
};

const obtenerFechaActual = () => {
  const now = new Date();
  return now.toLocaleDateString('es-MX', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  });
};

const generarFactura = () => {
  if (carrito.value.length > 0) {
    const doc = new jsPDF({ orientation: 'portrait', unit: 'mm', format: [80, 140 + carrito.value.length * 8] });
    const fecha = obtenerFechaActual();
    const total = obtenerTotalCarrito();
    // Ticket box
    doc.setDrawColor(60, 60, 60);
    doc.setLineWidth(0.8);
    doc.roundedRect(5, 5, 70, 130 + carrito.value.length * 8, 6, 6, 'S');
    // Logo
    doc.setFontSize(18);
    doc.setTextColor(44, 62, 80);
    doc.text('🍽️', 40, 18, { align: 'center' });
    // Título
    doc.setFontSize(14);
    doc.setTextColor(33, 33, 33);
    doc.text('RESTAURANT', 40, 26, { align: 'center' });
    doc.setFontSize(10);
    doc.setTextColor(100, 100, 100);
    doc.text('Factura', 40, 32, { align: 'center' });
    // Datos
    doc.setFontSize(8.5);
    doc.setTextColor(80, 80, 80);
    doc.text('Fecha:', 10, 40); doc.text(fecha, 25, 40);
    doc.text('Mesero:', 10, 45); doc.text(mesero.nombre, 25, 45);
    doc.text('ID:', 10, 50); doc.text(mesero.id, 25, 50);
    // Línea
    doc.setDrawColor(200, 200, 200);
    doc.line(10, 54, 70, 54);
    // Tabla encabezado
    doc.setFontSize(8.5);
    doc.setTextColor(33, 33, 33);
    doc.text('Producto', 10, 59);
    doc.text('Cant', 38, 59);
    doc.text('Precio', 48, 59);
    doc.text('Subt.', 62, 59);
    doc.setDrawColor(220, 220, 220);
    doc.line(10, 61, 70, 61);
    // Productos
    let y = 66;
    doc.setFontSize(8);
    doc.setTextColor(60, 60, 60);
    carrito.value.forEach((item) => {
      doc.text(item.nombre.length > 15 ? item.nombre.slice(0, 15) + '…' : item.nombre, 10, y);
      doc.text(item.cantidad.toString(), 38, y, { align: 'right' });
      doc.text('$' + item.precio.toFixed(2), 48, y, { align: 'right' });
      doc.text('$' + (item.precio * item.cantidad).toFixed(2), 68, y, { align: 'right' });
      y += 7;
    });
    // Línea
    doc.setDrawColor(200, 200, 200);
    doc.line(10, y, 70, y);
    // Total
    y += 6;
    doc.setFontSize(11);
    doc.setTextColor(33, 33, 33);
    doc.text('TOTAL:', 10, y);
    doc.setFont('helvetica', 'bold');
    doc.text('$' + total.toFixed(2), 68, y, { align: 'right' });
    doc.setFont('helvetica', 'normal');
    // Mensaje
    y += 12;
    doc.setFontSize(9);
    doc.setTextColor(44, 62, 80);
    doc.text('¡Gracias por su visita!', 40, y, { align: 'center' });
    // Guardar
    doc.save('factura-' + Date.now() + '.pdf');
    mostrarFactura.value = true;
    mostrarCarrito.value = false;
  }
};

const cerrarFactura = () => {
  despachosHoy.value += 1;
  carrito.value = [];
  mostrarFactura.value = false;
};

const abrirModalProducto = () => {
  nuevoProducto.nombre = '';
  nuevoProducto.descripcion = '';
  nuevoProducto.precio = 0;
  nuevoProducto.stock = 0;
  nuevoProducto.imagen = '';
  alertaProducto.value = '';
  mostrarModalProducto.value = true;
};

const guardarProducto = () => {
  // Validaciones
  if (!nuevoProducto.nombre.trim()) {
    alertaProducto.value = 'El nombre es obligatorio.';
    return;
  }
  if (!nuevoProducto.descripcion.trim()) {
    alertaProducto.value = 'La descripción es obligatoria.';
    return;
  }
  if (!nuevoProducto.imagen.trim() || !/^https?:\/\/.+\.(jpg|jpeg|png|webp|gif)$/i.test(nuevoProducto.imagen.trim())) {
    alertaProducto.value = 'La URL de la imagen es obligatoria y debe ser válida (.jpg, .png, .webp, .gif)';
    return;
  }
  if (!nuevoProducto.precio || nuevoProducto.precio <= 0) {
    alertaProducto.value = 'El precio debe ser mayor a 0.';
    return;
  }
  if (!nuevoProducto.stock || nuevoProducto.stock <= 0) {
    alertaProducto.value = 'El stock debe ser mayor a 0.';
    return;
  }
  // Guardar
  const nuevoId = productos.value.length > 0 
    ? Math.max(...productos.value.map(p => p.id)) + 1 
    : 1;
  productos.value.push({
    id: nuevoId,
    nombre: nuevoProducto.nombre,
    descripcion: nuevoProducto.descripcion,
    precio: nuevoProducto.precio,
    stock: nuevoProducto.stock,
    imagen: nuevoProducto.imagen,
    cantidadSeleccionada: 1
  });
  mostrarModalProducto.value = false;
  alertaProducto.value = '';
};

const eliminarProducto = (id) => {
  if (confirm('¿Estás seguro de eliminar este producto?')) {
    const index = productos.value.findIndex(p => p.id === id);
    if (index !== -1) {
      productos.value.splice(index, 1);
    }
  }
};
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.app-container {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: white;
  padding: 15px 25px;
  border-radius: 15px;
  margin-bottom: 20px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
}

.header h1 {
  color: #333;
  font-size: 1.5rem;
}

.despachos {
  margin-top: 5px;
  color: #666;
}

.header-right {
  display: flex;
  align-items: center;
  gap: 10px;
}

.btn-header {
  padding: 10px 20px;
  background: #28a745;
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: bold;
  transition: background 0.3s;
}

.btn-header:hover {
  background: #218838;
}

.btn-factura-header {
  background: #667eea;
}

.btn-factura-header:hover {
  background: #5568d3;
}

.btn-factura-header:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.perfil {
  display: flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
  padding: 8px 15px;
  border-radius: 25px;
  background: #f0f0f0;
  transition: background 0.3s;
}

.perfil:hover {
  background: #e0e0e0;
}

.perfil-info {
  display: flex;
  flex-direction: column;
  text-align: right;
}

.perfil-nombre {
  font-weight: bold;
  color: #333;
}

.perfil-id {
  font-size: 0.8rem;
  color: #666;
}

.perfil-foto {
  width: 45px;
  height: 45px;
  border-radius: 50%;
  border: 3px solid #667eea;
}

.btn-perfil {
  padding: 10px 20px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  font-size: 1rem;
  transition: background 0.3s;
}

.btn-perfil:hover {
  background: #5568d3;
}

.perfil-view {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 60vh;
}

.perfil-card {
  background: white;
  padding: 40px;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

.perfil-card h2 {
  margin-bottom: 20px;
  color: #333;
}

.perfil-grande {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  margin: 20px 0;
  border: 4px solid #667eea;
}

.perfil-card p {
  margin: 10px 0;
  font-size: 1.1rem;
  color: #555;
}

.btn-volver {
  margin-top: 20px;
  padding: 12px 30px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  font-size: 1rem;
}

.main-layout {
  display: flex;
  position: relative;
  min-height: calc(100vh - 100px);
}

.productos-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  flex: 1;
  padding-right: 20px;
  padding-bottom: 80px;
  padding-top: 50px;
}

.product-card {
  background: white;
  padding: 20px;
  border-radius: 15px;
  text-align: center;
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
  transition: transform 0.3s;
  position: relative;
}

.product-card:hover {
  transform: translateY(-5px);
}

.btn-eliminar-producto {
  position: absolute;
  top: 10px;
  right: 10px;
  background: #dc3545;
  color: white;
  border: none;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 0.9rem;
  transition: background 0.3s;
  z-index: 10;
}

.btn-eliminar-producto:hover {
  background: #c82333;
}

.product-image {
  width: 100%;
  height: 120px;
  object-fit: cover;
  border-radius: 10px;
  margin-bottom: 10px;
}
.alerta-producto {
  background: #ffdddd;
  color: #b71c1c;
  border: 1px solid #b71c1c;
  border-radius: 8px;
  padding: 10px;
  margin-bottom: 10px;
  text-align: center;
  font-weight: bold;
}

.product-card h3 {
  color: #333;
  margin-bottom: 8px;
}

.product-desc {
  color: #666;
  font-size: 0.9rem;
  margin-bottom: 10px;
}

.product-precio {
  font-size: 1.5rem;
  font-weight: bold;
  color: #667eea;
  margin-bottom: 5px;
}

.product-stock {
  color: #28a745;
  font-size: 0.9rem;
  margin-bottom: 15px;
}

.stock-bajo {
  color: #dc3545;
  font-weight: bold;
}

.product-actions {
  display: flex;
  gap: 10px;
  justify-content: center;
}

.cantidad-input {
  width: 60px;
  padding: 8px;
  border: 2px solid #ddd;
  border-radius: 8px;
  text-align: center;
  font-size: 1rem;
}

.btn-agregar {
  padding: 10px 15px;
  background: #28a745;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.3s;
}

.btn-agregar:hover:not(:disabled) {
  background: #218838;
}

.btn-agregar:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.carrito-lateral {
  position: fixed;
  right: -350px;
  top: 0;
  width: 350px;
  height: 100vh;
  background: white;
  box-shadow: -4px 0 20px rgba(0,0,0,0.2);
  transition: right 0.3s ease;
  z-index: 200;
  display: flex;
  flex-direction: column;
}

.carrito-lateral.abierto {
  right: 0;
}

.carrito-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  background: #667eea;
  color: white;
}

.carrito-header h3 {
  margin: 0;
}

.btn-cerrar-carrito {
  background: none;
  border: none;
  color: white;
  font-size: 1.5rem;
  cursor: pointer;
}

.carrito-vacio {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: #999;
}

.carrito-vacio-icon {
  font-size: 4rem;
  margin-top: 20px;
}

.carrito-items {
  flex: 1;
  overflow-y: auto;
  padding: 15px;
}

.carrito-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
  border-bottom: 1px solid #eee;
  background: #f9f9f9;
  border-radius: 8px;
  margin-bottom: 10px;
}

.carrito-item-info {
  display: flex;
  flex-direction: column;
}

.carrito-nombre {
  font-weight: bold;
  color: #333;
}

.carrito-precio {
  color: #666;
  font-size: 0.85rem;
}

.carrito-subtotal {
  color: #667eea;
  font-weight: bold;
  font-size: 0.9rem;
}

.btn-eliminar {
  background: #ff6b6b;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.3s;
}

.btn-eliminar:hover {
  background: #ee5a5a;
}

.carrito-footer {
  padding: 20px;
  border-top: 2px solid #667eea;
  background: #f8f9fa;
}

.carrito-total-final {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
  font-size: 1.3rem;
}

.carrito-total-final strong {
  color: #667eea;
  font-size: 1.5rem;
}

.btn-factura {
  width: 100%;
  padding: 15px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-size: 1.1rem;
  font-weight: bold;
  transition: background 0.3s;
}

.btn-factura:hover {
  background: #5568d3;
}

.modal-factura {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.factura-content {
  background: white;
  padding: 30px;
  border-radius: 15px;
  max-width: 500px;
  width: 90%;
  max-height: 80vh;
  overflow-y: auto;
}

.factura-content h2 {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
  border-bottom: 2px solid #667eea;
  padding-bottom: 10px;
}

.factura-header {
  margin-bottom: 20px;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 8px;
}

.factura-header p {
  margin: 5px 0;
  color: #555;
}

.factura-tabla {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.factura-tabla th,
.factura-tabla td {
  padding: 10px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.factura-tabla th {
  background: #667eea;
  color: white;
}

.factura-total {
  text-align: right;
  padding: 15px;
  background: #667eea;
  color: white;
  border-radius: 8px;
  margin-bottom: 20px;
}

.factura-footer {
  text-align: center;
}

.factura-footer p {
  color: #666;
  margin-bottom: 15px;
}

.pdf-mensaje {
  color: #28a745 !important;
  font-weight: bold;
  background: #d4edda;
  padding: 10px;
  border-radius: 8px;
  margin-bottom: 15px !important;
}

.btn-cerrar {
  padding: 12px 30px;
  background: #333;
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  font-size: 1rem;
  transition: background 0.3s;
}

.btn-cerrar:hover {
  background: #555;
}

/* Formulario de nuevo producto */
.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  color: #333;
  font-weight: bold;
}

.form-group input {
  width: 100%;
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
}

.form-group input:focus {
  outline: none;
  border-color: #667eea;
}

.emoji-selector {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.emoji-btn {
  width: 40px;
  height: 40px;
  font-size: 1.5rem;
  border: 2px solid #ddd;
  border-radius: 8px;
  background: white;
  cursor: pointer;
  transition: all 0.3s;
}

.emoji-btn:hover {
  border-color: #667eea;
}

.emoji-btn.selected {
  border-color: #667eea;
  background: #e8eaf6;
}

.modal-actions {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin-top: 20px;
}
</style>
