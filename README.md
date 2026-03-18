<!DOCTYPE html>
<html lang="am">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ማህበረ ቂርቆስ - 3D መዝገብ</title>
    
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+Ethiopic:wght@300;400;700&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

    <style>
        :root {
            --orthodox-red: #4a0404;
            --gold: #d4af37;
            --gold-grad: linear-gradient(135deg, #bf953f 0%, #fcf6ba 50%, #b38728 100%);
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; margin: 0; padding: 0; }

        body {
            background-color: #000;
            color: #f4ece1;
            font-family: 'Noto Sans Ethiopic', sans-serif;
            padding-bottom: env(safe-area-inset-bottom);
            overflow-x: hidden;
        }

        /* 3D MOTION BACKGROUND */
        .parallax-bg {
            position: fixed; top: -10%; left: -10%; width: 120%; height: 120%;
            z-index: -10;
            background-image: 
                linear-gradient(rgba(0,0,0,0.7), rgba(74, 4, 4, 0.85)), 
                url('saints.jpg'); /* Ensure your image of Gabriel, Kirkos & Eyeluta is named saints.jpg */
            background-size: cover;
            background-position: center;
            transition: transform 0.15s ease-out;
        }

        #canvas-3d { position: fixed; top: 0; left: 0; z-index: -5; pointer-events: none; opacity: 0.3; }

        .container { padding: 20px; max-width: 500px; margin: 0 auto; }

        header { 
            padding: 40px 0; text-align: center; 
            border-bottom: 2px double var(--gold); margin-bottom: 30px;
        }
        .logo { font-size: 2.3rem; font-weight: 700; background: var(--gold-grad); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }

        .section-h { color: var(--gold); text-align: center; margin: 30px 0 15px; font-size: 1.3rem; text-shadow: 0 2px 10px rgba(0,0,0,0.8); }

        /* Scrollable Table */
        .glass-card { 
            background: rgba(0, 0, 0, 0.7); 
            border: 1px solid var(--gold); 
            border-radius: 20px; padding: 10px; 
            backdrop-filter: blur(15px); margin-bottom: 25px;
            overflow-x: auto;
        }
        table { width: 100%; min-width: 420px; border-collapse: collapse; }
        th { color: var(--gold); padding: 12px; border-bottom: 2px solid var(--gold); text-align: left; font-size: 0.9rem; }
        td { padding: 12px; border-bottom: 1px solid rgba(212, 175, 55, 0.15); font-size: 0.85rem; }
        .current-month { background: rgba(212, 175, 55, 0.2); border-left: 4px solid var(--gold); }

        /* Members Grid */
        .mem-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
        .mem-item { 
            background: rgba(74, 4, 4, 0.4); padding: 20px 10px; border-radius: 20px; 
            text-align: center; border: 1px solid rgba(212, 175, 55, 0.3);
        }
        .mem-item:active { transform: scale(0.95); background: var(--orthodox-red); }

        .pfp-box { position: relative; width: 75px; height: 75px; margin: 0 auto 10px; }
        .pfp-img { width: 100%; height: 100%; border-radius: 50%; border: 2px solid var(--gold); object-fit: cover; background: #111; }

        /* Modal Phone UI */
        .modal { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.95); display: none; justify-content: center; align-items: center; z-index: 9999; }
        .modal-content { background: var(--orthodox-red); border: 2px solid var(--gold); padding: 35px 20px; border-radius: 25px; width: 85%; text-align: center; }
        .btn-stack { display: flex; flex-direction: column; gap: 15px; margin-top: 25px; }
        .btn { padding: 16px; border-radius: 15px; text-decoration: none; font-weight: bold; display: block; border: none; font-size: 1rem; }
        .btn-call { background: var(--gold-grad); color: #000; }
        .btn-sms { border: 1px solid var(--gold); color: var(--gold); background: transparent; }

        #broadcast-fab { 
            position: fixed; bottom: 30px; left: 50%; transform: translateX(-50%);
            background: var(--gold-grad); color: #000; padding: 18px 35px; 
            border-radius: 50px; font-weight: 900; z-index: 1000; border: none;
            box-shadow: 0 10px 30px rgba(0,0,0,0.8);
        }
    </style>
</head>
<body>

<div class="parallax-bg" id="bg3d"></div>
<canvas id="canvas-3d"></canvas>

<div class="container">
    <header>
        <div style="font-size:1.5rem; color:var(--gold); margin-bottom:5px;">✙</div>
        <h1 class="logo">ማህበረ ቂርቆስ</h1>
        <p style="color:var(--gold); letter-spacing: 2px;">ቅዱስ ገብርኤል • ቅዱስ ቂርቆስ • ቅድስት ኢየሉጣ</p>
    </header>

    <h2 class="section-h">የ፲፭ ዓመታዊ መርሃ ግብር</h2>
    <div class="glass-card">
        <table>
            <thead>
                <tr><th>ወር</th><th>መሪ (Leader)</th><th>ትምህርት (Teacher)</th></tr>
            </thead>
            <tbody id="rotTable"></tbody>
        </table>
    </div>

    <h2 class="section-h">አገልጋዮች (Members)</h2>
    <div class="mem-grid" id="memGrid"></div>
    <div style="height: 110px;"></div>
</div>

<button id="broadcast-fab" onclick="sendGroupSMS()">📢 መልዕክት ለሁሉም ላክ</button>

<div class="modal" id="modal" onclick="closeM()">
    <div class="modal-content" onclick="event.stopPropagation()">
        <img id="m-pfp" src="" style="width:120px; height:120px; border-radius:50%; border:3px solid var(--gold); object-fit:cover; margin-bottom:15px;">
        <h3 id="m-name" style="color:var(--gold); font-size: 1.5rem;"></h3>
        <p id="m-duty" style="color:#fff; opacity:0.8; font-size:0.9rem; margin-bottom:25px;"></p>
        <div class="btn-stack">
            <a id="link-call" href="" class="btn btn-call">📞 በስልክ ደውል</a>
            <a id="link-sms" href="" class="btn btn-sms">✉️ መልዕክት ላክ</a>
        </div>
        <button onclick="closeM()" style="margin-top:25px; background:none; border:none; color:var(--gold); opacity:0.5;">ተመለስ</button>
    </div>
</div>

<script>
    // THE FULL 8 MEMBERS
    const members = [
        { id: 1, name: "Yeabsera Tatek", phone: "0992657057", duty: "የመዝሙር ዋና ሃላፊ" },
        { id: 2, name: "Eyueal Yitagesu", phone: "0911941226", duty: "ትምህርት ክፍል ዋና ሃላፊ" },
        { id: 3, name: "Daniel Hailu", phone: "0976478403", duty: "የዝማሬ እና የዜማ አስተባባሪ" },
        { id: 4, name: "Haymanot Belayneh", phone: "0964451578", duty: "ምክትል ሊቀመንበር" },
        { id: 5, name: "Yibeltal Bitew", phone: "0968904286", duty: "የመዝሙር እና የዜማ አስተባባሪ" },
        { id: 6, name: "Bereket Tadesse", phone: "0972044406", duty: "ዋና ጸሃፊ" },
        { id: 7, name: "Nahom Nigusee", phone: "0947291809", duty: "አባላት ምግበረ ሰናይ ተቆጣጣሪ" },
        { id: 8, name: "Abraham Aneley", phone: "0926296280", duty: "ጉዞ እና ምግባረ ሰናይ አስተባባሪ" }
    ];

    // FULL 12-MONTH ROTATION FOR THE 15th
    const rotation = [
        { m: "መስከረም", p: "Yeabsera T.", t: "Eyueal Y." },
        { m: "ጥቅምት", p: "Daniel H.", t: "Haymanot B." },
        { m: "ህዳር", p: "Yibeltal B.", t: "Bereket T." },
        { m: "ታህሳስ", p: "Nahom N.", t: "Abraham A." },
        { m: "ጥር", p: "Eyueal Y.", t: "Daniel H." },
        { m: "የካቲት", p: "Haymanot B.", t: "Yeabsera T." },
        { m: "መጋቢት", p: "Bereket T.", t: "Yibeltal B." },
        { m: "ሚያዝያ", p: "Abraham A.", t: "Nahom N." },
        { m: "ግንቦት", p: "Yeabsera T.", t: "Daniel H." },
        { m: "ሰኔ", p: "Eyueal Y.", t: "Haymanot B." },
        { m: "ሐምሌ", p: "Yibeltal B.", t: "Bereket T." },
        { m: "ነሐሴ", p: "Nahom N.", t: "Abraham A." }
    ];

    function init() {
        const tb = document.getElementById('rotTable');
        const ethMonthIdx = (new Date().getMonth() + 4) % 12; // Approximation of Eth month

        rotation.forEach((r, i) => {
            const tr = document.createElement('tr');
            if(i === ethMonthIdx) tr.className = 'current-month';
            tr.innerHTML = `<td style="color:var(--gold); font-weight:700;">${r.m} ፲፭</td><td>${r.p}</td><td>${r.t}</td>`;
            tb.appendChild(tr);
        });

        const grid = document.getElementById('memGrid');
        members.forEach(m => {
            const pic = localStorage.getItem(`pfp_${m.id}`) || `https://ui-avatars.com/api/?name=${m.name}&background=4a0404&color=d4af37`;
            const div = document.createElement('div');
            div.className = 'mem-item';
            div.innerHTML = `<div class="pfp-box"><img src="${pic}" class="pfp-img"></div><div style="font-weight:700; font-size:0.8rem; color:var(--gold);">${m.name}</div>`;
            div.onclick = () => {
                document.getElementById('m-name').innerText = m.name;
                document.getElementById('m-duty').innerText = m.duty;
                document.getElementById('m-pfp').src = pic;
                document.getElementById('link-call').href = "tel:" + m.phone;
                document.getElementById('link-sms').href = "sms:" + m.phone;
                document.getElementById('modal').style.display = 'flex';
            };
            grid.appendChild(div);
        });
    }

    // 3D MOTION LOGIC: Tilt your phone to see the background move
    if (typeof DeviceOrientationEvent.requestPermission === 'function') {
        // iOS requires permission for motion
        document.body.onclick = () => {
            DeviceOrientationEvent.requestPermission();
        };
    }

    window.addEventListener('deviceorientation', (e) => {
        const moveX = e.gamma / 3; // side to side
        const moveY = e.beta / 3;  // front to back
        document.getElementById('bg3d').style.transform = `translate(${-moveX}px, ${-moveY}px) scale(1.15)`;
    });

    function closeM() { document.getElementById('modal').style.display = 'none'; }

    function sendGroupSMS() {
        const isIOS = /iPhone|iPad|iPod/i.test(navigator.userAgent);
        const nums = members.map(m => m.phone).join(isIOS ? ',' : ',');
        window.location.href = `sms:${nums}${isIOS ? '&' : '?'}body=${encodeURIComponent("በስመ አብ ወወልድ ወመንፈስ ቅዱስ አሐዱ አምላክ አሜን። ለመርሃ ግብሩ ዝግጁ ሁኑ።")}`;
    }

    // 3D Particle Scene
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({ canvas: document.getElementById('canvas-3d'), alpha: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.position.z = 50;
    const mesh = new THREE.Mesh(new THREE.OctahedronGeometry(15, 0), new THREE.MeshStandardMaterial({ color: 0xd4af37, wireframe: true }));
    scene.add(mesh, new THREE.AmbientLight(0xffffff, 2));
    function anim() { requestAnimationFrame(anim); mesh.rotation.y += 0.005; renderer.render(scene, camera); }

    init(); anim();
</script>
</body>
</html>
