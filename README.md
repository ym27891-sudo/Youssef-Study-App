<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>مساعد يوسف الذكي</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { background: #f0f4f8; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        .card { background: white; border-radius: 20px; box-shadow: 0 10px 25px rgba(0,0,0,0.05); border-top: 5px solid #667eea; margin-bottom: 20px; }
        .btn-grad { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; font-weight: bold; transition: 0.3s; }
        .btn-grad:active { transform: scale(0.95); }
    </style>
</head>
<body class="p-4">
    <div class="max-w-md mx-auto">
        <h1 class="text-2xl font-bold text-center text-gray-800 mb-6">🎓 مساعد المذاكرة | يوسف محمد</h1>
        
        <!-- إضافة مادة -->
        <div class="card p-6">
            <h2 class="font-bold mb-4 text-gray-700">📦 إضافة مادة جديدة</h2>
            <input id="name" type="text" placeholder="اسم المادة (إدارة، تسويق...)" class="w-full p-3 mb-3 border rounded-xl focus:ring-2 focus:ring-blue-400 outline-none">
            <input id="pgs" type="number" placeholder="عدد الصفحات" class="w-full p-3 mb-3 border rounded-xl focus:ring-2 focus:ring-blue-400 outline-none">
            <input id="dt" type="date" class="w-full p-3 mb-4 border rounded-xl focus:ring-2 focus:ring-blue-400 outline-none">
            <button onclick="add()" class="btn-grad w-full py-3 rounded-xl shadow-lg">حفظ المادة في الـ Inventory ➕</button>
        </div>

        <!-- منبه التركيز -->
        <div class="card p-6 text-center">
            <h2 class="font-bold mb-4 text-gray-700">⏱️ منبه التركيز (Pomodoro)</h2>
            <div id="disp" class="text-5xl font-bold text-blue-600 mb-4">25:00</div>
            <button id="startBtn" onclick="startT()" class="w-full bg-green-500 text-white py-3 rounded-xl font-bold hover:bg-green-600 shadow-md transition">ابدأ جلسة المذاكرة ▶️</button>
        </div>

        <!-- الجدول الذكي AI -->
        <div class="card p-6 border-t-5 border-purple-500">
            <h2 class="font-bold mb-4 text-purple-700">🪄 جدولك المريح (AI)</h2>
            <p class="text-xs text-gray-500 mb-3 text-center">الذكاء الاصطناعي هيقسم لك صفحاتك على الأيام الفاضلة</p>
            <button onclick="askAI()" class="w-full bg-gradient-to-r from-purple-500 to-blue-500 text-white py-3 rounded-xl font-bold shadow-lg">ولد لي جدولاً ذكياً الآن</button>
            <div id="aiRes" class="mt-4 text-gray-700 text-sm leading-relaxed whitespace-pre-line bg-gray-50 p-3 rounded-lg border hidden"></div>
        </div>

        <!-- قائمة المواد المضافة -->
        <div id="list" class="space-y-4 pb-10"></div>
    </div>

    <audio id="bell" src="https://www.soundjay.com/misc/sounds/bell-ringing-05.mp3"></audio>

    <script>
        let subs = JSON.parse(localStorage.getItem('y_study_data')) || [];
        
        function add() {
            const n = document.getElementById('name').value, p = document.getElementById('pgs').value, d = document.getElementById('dt').value;
            if(!n || !p || !d) return alert('كمل البيانات يا وحش عشان نعرف نحسبلك!');
            subs.push({n, p, d}); 
            localStorage.setItem('y_study_data', JSON.stringify(subs)); 
            document.getElementById('name').value = '';
            document.getElementById('pgs').value = '';
            render();
        }

        function render() {
            const l = document.getElementById('list'); l.innerHTML = '';
            if(subs.length > 0) l.innerHTML = '<h3 class="font-bold text-gray-600 mb-2">موادك الحالية:</h3>';
            subs.forEach((s, index) => {
                const days = Math.ceil((new Date(s.d) - new Date())/(1000*60*60*24));
                l.innerHTML += `
                    <div class="card p-4 border-r-8 border-blue-500 flex justify-between items-center">
                        <div>
                            <b class="text-blue-800">${s.n}</b><br>
                            <span class="text-xs text-gray-600">📄 ${s.p} صفحة | 📅 الامتحان بعد ${days > 0 ? days : 0} يوم</span>
                        </div>
                        <button onclick="del(${index})" class="text-red-400 text-xs">حذف</button>
                    </div>`;
            });
        }

        function del(i) {
            subs.splice(i, 1);
            localStorage.setItem('y_study_data', JSON.stringify(subs));
            render();
        }

        function startT() {
            let s = 25 * 60;
            const btn = document.getElementById('startBtn'); 
            btn.disabled = true;
            btn.classList.add('opacity-50');
            const i = setInterval(() => {
                const m = Math.floor(s/60), sec = s%60;
                document.getElementById('disp').innerText = `${m}:${String(sec).padStart(2,'0')}`;
                if(s-- <= 0) { 
                    clearInterval(i); 
                    document.getElementById('bell').play(); 
                    alert('عاش يا يوسف! خلصت 25 دقيقة تركيز. خد بريك 5 دقايق.'); 
                    btn.disabled = false;
                    btn.classList.remove('opacity-50');
                    document.getElementById('disp').innerText = "25:00";
                }
            }, 1000);
        }

        async function askAI() {
            if(subs.length === 0) return alert('ضيف مادة الأول في الـ Inventory يا يوسف!');
            const resBox = document.getElementById('aiRes');
            resBox.classList.remove('hidden');
            resBox.innerText = "جاري الاتصال بـ Gemini... ثواني يا بطل.";
            
            try {
                const prompt = `أنا يوسف، طالب خريج تجارة وبذاكر حالياً. عندي المواد دي: ${JSON.stringify(subs)}. اعملي جدول مذاكرة مريح جداً بلهجة مصرية، وزعلي الصفحات على الأيام المتبقية بذكاء عشان ما زهقش، واديني نصيحة تشجيعية في الآخر.`;
                    method: 'POST', 
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({contents: [{parts: [{text: prompt}]}]})
                });
                const data = await response.json();
                resBox.innerText = data.candidates[0].content.parts[0].text;
            } catch(e) { 
                resBox.innerText = "الخدمة عليها ضغط، دوس تاني وهتشتغل معاك."; 
            }
        }
        render();
    </script>
</body>
</html>
