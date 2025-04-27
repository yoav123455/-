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
