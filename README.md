# -<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>מכירת חצר וירטואלית</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header>
        <h1>מכירת חצר וירטואלית</h1>
    </header>
    <main>
        <div id="productList" class="products-container">
            <!-- מוצרים יתווספו כאן דינמית -->
        </div>
    </main>
    <footer>
        <p><a href="admin.html">ניהול האתר (דרושה סיסמה)</a></p>
    </footer>
    <script>
        // קריאת רשימת המוצרים מ-LocalStorage (או מערך ריק אם אין)
        let products = JSON.parse(localStorage.getItem('products') || '[]');

        function renderProducts() {
            const list = document.getElementById('productList');
            list.innerHTML = '';
            if (products.length === 0) {
                // אם אין מוצרים, תוצג הודעה מתאימה
                list.innerHTML = '<p>אין מוצרים להצגה</p>';
                return;
            }
            // מעבר על כל מוצר ויצירת כרטיס תצוגה
            products.forEach((product) => {
                const card = document.createElement('div');
                card.className = 'product-card';
                if (product.reserved) {
                    card.classList.add('reserved');
                }
                // תמונה
                const img = document.createElement('img');
                img.src = product.image || '';
                img.alt = product.name;
                card.appendChild(img);
                // שם המוצר
                const nameElem = document.createElement('h3');
                nameElem.textContent = product.name;
                card.appendChild(nameElem);
                // תיאור המוצר
                const descElem = document.createElement('p');
                descElem.textContent = product.description;
                card.appendChild(descElem);
                // מחיר
                const priceElem = document.createElement('p');
                priceElem.className = 'price';
                priceElem.textContent = product.price + ' ₪';
                card.appendChild(priceElem);
                // כפתור "שריין"
                const reserveBtn = document.createElement('button');
                reserveBtn.textContent = product.reserved ? 'נשמר' : 'שריין';
                reserveBtn.disabled = product.reserved;
                reserveBtn.onclick = () => {
                    // סימון הפריט כ"שמור" ועדכון ה-LocalStorage
                    product.reserved = true;
                    localStorage.setItem('products', JSON.stringify(products));
                    alert('הפריט שוריין!');
                    // פתיחת WhatsApp עם פרטי המוצר
                    const message = encodeURIComponent(`שלום, ברצוני להזמין את הפריט: ${product.name} במחיר ${product.price} ₪`);
                    window.open('https://api.whatsapp.com/send?text=' + message, '_blank');
                    renderProducts();
                };
                card.appendChild(reserveBtn);
                list.appendChild(card);
            });
        }

        // הצגת המוצרים עם טעינת הדף
        renderProducts();
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>ניהול אתר מכירת חצר</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header>
        <h1>ניהול מכירת חצר</h1>
    </header>
    <main>
        <!-- טופס התחברות -->
        <div id="loginSection">
            <h2>התחברות למערכת הניהול</h2>
            <label for="passInput">סיסמה:</label>
            <input type="password" id="passInput">
            <button id="loginBtn">התחבר</button>
            <p id="loginMsg" class="error"></p>
        </div>
        <!-- ממשק ניהול -->
        <div id="adminPanel" style="display:none;">
            <h2>הוספת / עריכת מוצר</h2>
            <form id="productForm">
                <!-- שדה נסתר למעקב אחרי עריכה -->
                <input type="hidden" id="editId" value="">
                <label for="nameInput">שם המוצר:</label>
                <input type="text" id="nameInput" required><br>
                <label for="descInput">תיאור:</label>
                <textarea id="descInput"></textarea><br>
                <label for="priceInput">מחיר:</label>
                <input type="number" id="priceInput" required><br>
                <label for="imageInput">תמונה:</label>
                <input type="file" id="imageInput" accept="image/*"><br>
                <button type="submit" id="saveBtn">שמור מוצר</button>
                <button type="button" id="cancelEditBtn" style="display:none;">ביטול</button>
            </form>

            <h2>מוצרים קיימים</h2>
            <table id="productsTable">
                <thead>
                    <tr><th>תמונה</th><th>שם</th><th>תיאור</th><th>מחיר</th><th>פעולות</th></tr>
                </thead>
                <tbody>
                    <!-- רשימת מוצרים תתווסף כאן -->
                </tbody>
            </table>

            <h2>שינוי סיסמה</h2>
            <form id="passwordForm">
                <label for="currentPass">סיסמה נוכחית:</label>
                <input type="password" id="currentPass" required><br>
                <label for="newPass">סיסמה חדשה:</label>
                <input type="password" id="newPass" required><br>
                <button type="submit">שנה סיסמה</button>
                <p id="passMsg" class="error"></p>
            </form>
        </div>
    </main>
    <script>
        // הגדרת הסיסמה (משתמשים בקידוד פשוט ללא הצפנה מתוחכמת)
        let adminPass = localStorage.getItem('adminPass') || '1234';

        const loginBtn = document.getElementById('loginBtn');
        const passInput = document.getElementById('passInput');
        const loginMsg = document.getElementById('loginMsg');
        const loginSection = document.getElementById('loginSection');
        const adminPanel = document.getElementById('adminPanel');

        // אירוע לכניסה: בדיקה מול הסיסמה השמורה
        loginBtn.onclick = () => {
            const entered = passInput.value;
            if (entered === adminPass) {
                loginSection.style.display = 'none';
                adminPanel.style.display = 'block';
                renderProductTable();
            } else {
                loginMsg.textContent = 'סיסמה שגויה';
            }
        };

        // קריאת רשימת המוצרים (אותה מבנה כמו ב-index.html)
        let products = JSON.parse(localStorage.getItem('products') || '[]');

        const productForm = document.getElementById('productForm');
        const editIdInput = document.getElementById('editId');
        const nameInput = document.getElementById('nameInput');
        const descInput = document.getElementById('descInput');
        const priceInput = document.getElementById('priceInput');
        const imageInput = document.getElementById('imageInput');
        const cancelEditBtn = document.getElementById('cancelEditBtn');

        // טיפול בשליחת טופס הוספת/עריכת מוצר
        productForm.onsubmit = function(e) {
            e.preventDefault();
            const name = nameInput.value.trim();
            const desc = descInput.value.trim();
            const price = priceInput.value;
            // אם נבחרה תמונה, קוראים אותה וממשיכים ל-saveProduct
            if (imageInput.files.length > 0) {
                const reader = new FileReader();
                reader.onload = () => {
                    const imgData = reader.result;
                    saveProduct(name, desc, price, imgData);
                };
                reader.readAsDataURL(imageInput.files[0]);
            } else {
                // אין תמונה חדשה
                saveProduct(name, desc, price, null);
            }
        };

        // שמירה של מוצר חדש או עריכה של מוצר קיים
        function saveProduct(name, desc, price, imgData) {
            const editId = editIdInput.value;
            if (editId) {
                // עריכת מוצר קיים
                const index = products.findIndex(p => p.id == editId);
                if (index !== -1) {
                    products[index].name = name;
                    products[index].description = desc;
                    products[index].price = price;
                    if (imgData) {
                        products[index].image = imgData;
                    }
                }
            } else {
                // הוספת מוצר חדש
                const newProduct = {
                    id: Date.now(),
                    name: name,
                    description: desc,
                    price: price,
                    image: imgData || '',
                    reserved: false
                };
                products.push(newProduct);
            }
            // עדכון ב-LocalStorage ופתיחת הטופס מחדש
            localStorage.setItem('products', JSON.stringify(products));
            productForm.reset();
            editIdInput.value = '';
            cancelEditBtn.style.display = 'none';
            renderProductTable();
        }

        // פונקציה להצגת טבלת המוצרים הקיימים
        function renderProductTable() {
            const tableBody = document.querySelector('#productsTable tbody');
            tableBody.innerHTML = '';
            if (products.length === 0) {
                tableBody.innerHTML = '<tr><td colspan="5">אין מוצרים</td></tr>';
                return;
            }
            products.forEach((prod) => {
                const row = document.createElement('tr');
                // תמונה (קטנה) במידה וקיימת
                const imgTd = document.createElement('td');
                if (prod.image) {
                    const img = document.createElement('img');
                    img.src = prod.image;
                    img.alt = prod.name;
                    img.width = 50;
                    imgTd.appendChild(img);
                }
                row.appendChild(imgTd);
                // שם, תיאור, מחיר
                ['name', 'description', 'price'].forEach(key => {
                    const td = document.createElement('td');
                    td.textContent = prod[key] || '';
                    row.appendChild(td);
                });
                // עמודת פעולות (עריכה ומחיקה)
                const actionsTd = document.createElement('td');
                const editBtn = document.createElement('button');
                editBtn.textContent = 'ערוך';
                editBtn.onclick = () => editProduct(prod.id);
                const deleteBtn = document.createElement('button');
                deleteBtn.textContent = 'מחק';
                deleteBtn.onclick = () => deleteProduct(prod.id);
                actionsTd.appendChild(editBtn);
                actionsTd.appendChild(deleteBtn);
                row.appendChild(actionsTd);

                tableBody.appendChild(row);
            });
        }

        // מילוי הטופס בפרטי מוצר קיים לעריכה
        function editProduct(id) {
            const prod = products.find(p => p.id == id);
            if (!prod) return;
            nameInput.value = prod.name;
            descInput.value = prod.description;
            priceInput.value = prod.price;
            editIdInput.value = id;
            cancelEditBtn.style.display = 'inline-block';
        }

        // מחיקת מוצר
        function deleteProduct(id) {
            if (confirm('למחוק מוצר זה?')) {
                products = products.filter(p => p.id != id);
                localStorage.setItem('products', JSON.stringify(products));
                renderProductTable();
            }
        }

        // כפתור "ביטול" בעמוד עריכה: מנקה את הטופס
        cancelEditBtn.onclick = function() {
            productForm.reset();
            editIdInput.value = '';
            cancelEditBtn.style.display = 'none';
        };

        // טיפול בטופס שינוי סיסמה
        const passwordForm = document.getElementById('passwordForm');
        const currentPass = document.getElementById('currentPass');
        const newPass = document.getElementById('newPass');
        const passMsg = document.getElementById('passMsg');

        passwordForm.onsubmit = function(e) {
            e.preventDefault();
            if (currentPass.value !== adminPass) {
                passMsg.textContent = 'הסיסמה הנוכחית שגויה';
                return;
            }
            // עדכון הסיסמה ב-LocalStorage
            adminPass = newPass.value;
            localStorage.setItem('adminPass', adminPass);
            passMsg.textContent = 'הסיסמה שונתה בהצלחה';
            passwordForm.reset();
        };
    </script>
</body>
</html>

body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f4f4f4;
    color: #333;
    direction: rtl;
}

header {
    background-color: #4CAF50;
    color: white;
    padding: 20px 10px;
    text-align: center;
}

h1, h2 {
    margin: 10px 0;
}

main {
    padding: 20px;
}

.products-container {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    justify-content: flex-start;
}

.product-card {
    background: white;
    border: 1px solid #ddd;
    border-radius: 8px;
    width: 200px;
    padding: 10px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    text-align: center;
}

.product-card.reserved {
    opacity: 0.6;
}

.product-card img {
    max-width: 100%;
    height: auto;
    border-radius: 4px;
}

.product-card h3 {
    margin: 10px 0 5px 0;
    font-size: 18px;
}

.product-card p {
    margin: 5px 0;
}

.product-card .price {
    font-weight: bold;
    margin-top: 5px;
}

.product-card button {
    margin-top: 10px;
    padding: 8px 12px;
    background-color: #4CAF50;
    border: none;
    border-radius: 4px;
    color: white;
    cursor: pointer;
}

.product-card button:disabled {
    background-color: #999;
    cursor: default;
}

footer {
    text-align: center;
    margin: 20px 0;
}

.error {
    color: red;
}

table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 10px;
}

table th, table td {
    border: 1px solid #ccc;
    padding: 5px;
}

table th {
    background-color: #eee;
}

button {
    padding: 6px 10px;
    margin: 2px;
    border: none;
    border-radius: 4px;
    background-color: #4CAF50;
    color: #fff;
    cursor: pointer;
}

button:hover {
    background-color: #45a049;
}

input[type=text], input[type=password], input[type=number], textarea {
    padding: 5px;
    width: 100%;
    box-sizing: border-box;
    margin: 2px 0 8px 0;
    border: 1px solid #ccc;
    border-radius: 4px;
}

textarea {
    resize: vertical;
}

input[type=file] {
    margin: 5px 0 10px 0;
}

label {
    font-weight: bold;
}

#loginSection, #adminPanel {
    max-width: 500px;
    margin: auto;
}

#loginSection input {
    width: calc(100% - 80px);
    display: inline-block;
}

#loginSection button {
    display: inline-block;
    padding: 6px 12px;
}

