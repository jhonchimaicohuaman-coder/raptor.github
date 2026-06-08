// Listado inicial con tus 20 productos provistos
const initialProducts = [
    { id: "1", name: "Arroz Extra", category: "Abarrotes", quality: "Alta", content: "1 kg", price: 5.50, expiry: "15/12/2027" },
    { id: "2", name: "Azúcar Rubia", category: "Abarrotes", quality: "Alta", content: "1 kg", price: 4.80, expiry: "20/10/2027" },
    { id: "3", name: "Aceite Vegetal", category: "Abarrotes", quality: "Premium", content: "1 L", price: 10.50, expiry: "30/09/2027" },
    { id: "4", name: "Leche Evaporada", category: "Lácteos", quality: "Alta", content: "400 g", price: 4.20, expiry: "15/03/2027" },
    { id: "5", name: "Yogurt Fresa", category: "Lácteos", quality: "Alta", content: "1 L", price: 7.50, expiry: "20/08/2026" },
    { id: "6", name: "Queso Fresco", category: "Lácteos", quality: "Media", content: "500 g", price: 12.00, expiry: "18/07/2026" },
    { id: "7", name: "Pan Integral", category: "Panadería", quality: "Alta", content: "600 g", price: 8.50, expiry: "15/06/2026" },
    { id: "8", name: "Galletas de Vainilla", category: "Snacks", quality: "Alta", content: "300 g", price: 3.80, expiry: "10/01/2027" },
    { id: "9", name: "Atún en Conserva", category: "Conservas", quality: "Premium", content: "170 g", price: 6.50, expiry: "15/05/2028" },
    { id: "10", name: "Fideos Espagueti", category: "Abarrotes", quality: "Alta", content: "500 g", price: 3.50, expiry: "30/11/2027" },
    { id: "11", name: "Café Instantáneo", category: "Bebidas", quality: "Premium", content: "200 g", price: 15.00, expiry: "25/02/2028" },
    { id: "12", name: "Agua Mineral", category: "Bebidas", quality: "Alta", content: "625 ml", price: 2.00, expiry: "15/01/2028" },
    { id: "13", name: "Gaseosa Cola", category: "Bebidas", quality: "Alta", content: "3 L", price: 11.50, expiry: "20/12/2026" },
    { id: "14", name: "Jugo de Naranja", category: "Bebidas", quality: "Alta", content: "1 L", price: 6.80, expiry: "15/09/2026" },
    { id: "15", name: "Chocolate en Barra", category: "Dulces", quality: "Premium", content: "100 g", price: 4.50, expiry: "30/10/2027" },
    { id: "16", name: "Detergente en Polvo", category: "Limpieza", quality: "Alta", content: "1 kg", price: 12.50, expiry: "01/05/2029" },
    { id: "17", name: "Jabón de Tocador", category: "Higiene", quality: "Alta", content: "125 g", price: 2.50, expiry: "15/07/2029" },
    { id: "18", name: "Shampoo Familiar", category: "Higiene", quality: "Premium", content: "750 ml", price: 18.00, expiry: "20/11/2028" },
    { id: "19", name: "Papel Higiénico (4 rollos)", category: "Higiene", quality: "Alta", content: "Pack", price: 9.00, expiry: "No aplica" },
    { id: "20", name: "Huevos de Gallina", category: "Perecibles", quality: "Alta", content: "Docena", price: 9.50, expiry: "20/06/2026" }
];

// Cargar del localStorage o usar la lista inicial por defecto
let products = JSON.parse(localStorage.getItem('corp_inventory')) || initialProducts;
if(!localStorage.getItem('corp_inventory')) {
    localStorage.setItem('corp_inventory', JSON.stringify(products));
}

let isEditing = false;

// Referencias del DOM
const productForm = document.getElementById('product-form');
const productIdInput = document.getElementById('product-id');
const productNameInput = document.getElementById('product-name');
const productCategoryInput = document.getElementById('product-category');
const productQualityInput = document.getElementById('product-quality');
const productContentInput = document.getElementById('product-content');
const productPriceInput = document.getElementById('product-price');
const productExpiryInput = document.getElementById('product-expiry');

const tableBody = document.getElementById('table-body');
const emptyState = document.getElementById('empty-state');
const totalProductsEl = document.getElementById('total-products');
const formTitle = document.getElementById('form-title');
const btnSubmit = document.getElementById('btn-submit');
const btnCancel = document.getElementById('btn-cancel');
const toast = document.getElementById('toast');

document.addEventListener('DOMContentLoaded', () => {
    renderProducts();
    productForm.addEventListener('submit', handleFormSubmit);
    btnCancel.addEventListener('click', resetForm);
});

function syncStorage() {
    localStorage.setItem('corp_inventory', JSON.stringify(products));
    renderProducts();
}

function renderProducts() {
    tableBody.innerHTML = '';
    
    if (products.length === 0) {
        emptyState.classList.remove('hidden');
    } else {
        emptyState.classList.add('hidden');
        
        products.forEach(product => {
            const tr = document.createElement('tr');
            
            // Clase CSS para badges según calidad
            const qClass = product.quality.toLowerCase();
            
            tr.innerHTML = `
                <td><small class="text-muted">#${product.id}</small></td>
                <td><strong>${product.name}</strong></td>
                <td>${product.category}</td>
                <td><span class="badge-q ${qClass}">${product.quality}</span></td>
                <td>${product.content}</td>
                <td>S/. ${parseFloat(product.price).toFixed(2)}</td>
                <td><span class="expiry-text">${product.expiry}</span></td>
                <td class="text-center">
                    <button class="btn-icon edit" onclick="editProduct('${product.id}')" title="Editar">
                        <span class="material-icons-round">edit</span>
                    </button>
                    <button class="btn-icon delete" onclick="deleteProduct('${product.id}')" title="Eliminar">
                        <span class="material-icons-round">delete</span>
                    </button>
                </td>
            `;
            tableBody.appendChild(tr);
        });
    }
    totalProductsEl.textContent = products.length;
}

function handleFormSubmit(e) {
    e.preventDefault();
    
    const productData = {
        id: productIdInput.value || (Math.max(...products.map(p => parseInt(p.id) || 0), 0) + 1).toString(),
        name: productNameInput.value.trim(),
        category: productCategoryInput.value,
        quality: productQualityInput.value,
        content: productContentInput.value.trim(),
        price: parseFloat(productPriceInput.value),
        expiry: productExpiryInput.value.trim()
    };

    if (isEditing) {
        products = products.map(p => p.id === productData.id ? productData : p);
        showToast('Producto actualizado exitosamente');
    } else {
        products.push(productData);
        showToast('Producto añadido al inventario');
    }

    syncStorage();
    resetForm();
}

function editProduct(id) {
    const product = products.find(p => p.id === id);
    if (!product) return;

    isEditing = true;
    formTitle.textContent = 'Modificar Artículo';
    btnSubmit.innerHTML = `<span class="material-icons-round">edit</span> Aplicar Cambios`;
    btnCancel.classList.remove('hidden');

    productIdInput.value = product.id;
    productNameInput.value = product.name;
    productCategoryInput.value = product.category;
    productQualityInput.value = product.quality;
    productContentInput.value = product.content;
    productPriceInput.value = product.price;
    productExpiryInput.value = product.expiry;
    
    productNameInput.focus();
}

function deleteProduct(id) {
    if (confirm('¿Retirar este producto de la lista definitivamente?')) {
        products = products.filter(p => p.id !== id);
        showToast('Producto removido');
        syncStorage();
        if (productIdInput.value === id) resetForm();
    }
}

function resetForm() {
    isEditing = false;
    productForm.reset();
    productIdInput.value = '';
    formTitle.textContent = 'Registrar Producto';
    btnSubmit.innerHTML = `<span class="material-icons-round">save</span> Guardar Producto`;
    btnCancel.classList.add('hidden');
}

function showToast(message) {
    toast.textContent = message;
    toast.classList.add('show');
    setTimeout(() => toast.classList.remove('show'), 3000);
}
