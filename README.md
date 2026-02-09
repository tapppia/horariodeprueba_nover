<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calendario Académico - Astrofísica</title>
    <link href="https://fonts.googleapis.com/css2?family=Oswald:wght@300;400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #FFD60A;
            --text: #333;
            --bg-body: #ffffff;
            --danger: #ff4444;
            --warning: #ffbb33;
            --success: #00C851;
        }

        body {
            font-family: 'Oswald', sans-serif;
            background-color: var(--bg-body);
            color: var(--text);
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .main-container {
            max-width: 900px;
            width: 100%;
            display: grid;
            grid-template-columns: 2fr 1fr; /* Calendario | Lista */
            gap: 30px;
        }

        /* --- CALENDARIO --- */
        .calendar-box {
            background: white;
            border-radius: 20px;
            padding: 25px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.08);
        }

        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }

        .month-title {
            font-size: 1.8rem;
            text-transform: uppercase;
            font-weight: 700;
            color: var(--primary);
            text-shadow: 1px 1px 0px #ddd;
        }

        .nav-btn {
            background: #333;
            color: white;
            border: none;
            padding: 5px 15px;
            border-radius: 50px;
            cursor: pointer;
            font-family: 'Oswald';
            font-size: 1.2rem;
        }
        .nav-btn:hover { background: var(--primary); color: #333; }

        .grid-days {
            display: grid;
            grid-template-columns: repeat(7, 1fr);
            gap: 5px;
            text-align: center;
        }

        .day-name {
            font-weight: bold;
            color: #888;
            padding-bottom: 10px;
        }

        .day-cell {
            height: 60px;
            border-radius: 10px;
            border: 1px solid #f0f0f0;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: 0.2s;
            position: relative;
        }

        .day-cell:hover { background-color: #fffde7; border-color: var(--primary); }
        
        .day-cell.today {
            border: 2px solid var(--primary);
            font-weight: bold;
        }

        .day-cell.has-event {
            background-color: var(--primary);
            color: white;
            border: none;
        }

        .event-dot {
            height: 6px;
            width: 6px;
            background-color: white;
            border-radius: 50%;
            margin-top: 4px;
            display: none;
        }
        .has-event .event-dot { display: block; }

        /* --- LISTA DE CUENTA REGRESIVA --- */
        .events-panel {
            background: #f9f9f9;
            border-radius: 20px;
            padding: 25px;
            border: 2px solid var(--primary);
        }

        h2 { margin-top: 0; text-transform: uppercase; font-size: 1.4rem; }

        .event-card {
            background: white;
            padding: 15px;
            border-radius: 12px;
            margin-bottom: 10px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.05);
            border-left: 5px solid #ccc;
            animation: popIn 0.3s ease;
        }

        @keyframes popIn { from{opacity:0; transform:translateY(10px)} to{opacity:1; transform:translateY(0)} }

        .days-left {
            font-size: 1.5rem;
            font-weight: bold;
            display: block;
        }
        .event-title { font-size: 1rem; color: #555; }
        .event-date { font-size: 0.8rem; color: #999; }

        /* Colores de urgencia */
        .urgent { border-left-color: var(--danger); color: var(--danger); }
        .soon { border-left-color: var(--warning); color: #d48806; }
        .chill { border-left-color: var(--success); color: var(--success); }

        @media (max-width: 768px) {
            .main-container { grid-template-columns: 1fr; }
        }
    </style>
</head>
<body>

<div class="main-container">
    
    <div class="calendar-box">
        <div class="header">
            <button class="nav-btn" onclick="changeMonth(-1)">&#8249;</button>
            <div class="month-title" id="monthDisplay"></div>
            <button class="nav-btn" onclick="changeMonth(1)">&#8250;</button>
        </div>

        <div class="grid-days" id="calendarGrid">
            </div>
        <p style="text-align: center; color: #999; font-size: 0.9rem; margin-top: 15px;">
            Haz clic en un día para agregar o borrar eventos.
        </p>
    </div>

    <div class="events-panel">
        <h2>⏳ Próximas Evaluaciones</h2>
        <div id="countdownList">
            <p style="color:#777">No hay eventos próximos.</p>
        </div>
    </div>

</div>

<script>
    let nav = 0; // Navegación de meses
    let clicked = null;
    let events = localStorage.getItem('misEventos') ? JSON.parse(localStorage.getItem('misEventos')) : {};

    const calendar = document.getElementById('calendarGrid');
    const monthDisplay = document.getElementById('monthDisplay');
    const countdownList = document.getElementById('countdownList');
    const diasSemana = ['Dom', 'Lun', 'Mar', 'Mié', 'Jue', 'Vie', 'Sáb'];

    function load() {
        const dt = new Date();

        if (nav !== 0) {
            dt.setMonth(new Date().getMonth() + nav);
        }

        const day = dt.getDate();
        const month = dt.getMonth();
        const year = dt.getFullYear();

        const firstDayOfMonth = new Date(year, month, 1);
        const daysInMonth = new Date(year, month + 1, 0).getDate();
        
        const dateString = firstDayOfMonth.toLocaleDateString('es-ES', {
            weekday: 'long',
            year: 'numeric',
            month: 'long',
        });
        const paddingDays = firstDayOfMonth.getDay();

        monthDisplay.innerText = `${dt.toLocaleDateString('es-ES', { month: 'long' })} ${year}`;

        calendar.innerHTML = '';

        // Cabecera de días
        diasSemana.forEach(dia => {
            const div = document.createElement('div');
            div.classList.add('day-name');
            div.innerText = dia;
            calendar.appendChild(div);
        });

        // Días vacíos previos
        for(let i = 1; i <= paddingDays; i++) {
            const daySquare = document.createElement('div');
            daySquare.classList.add('day-cell');
            daySquare.style.visibility = 'hidden';
            calendar.appendChild(daySquare);
        }

        // Días del mes
        for(let i = 1; i <= daysInMonth; i++) {
            const daySquare = document.createElement('div');
            daySquare.classList.add('day-cell');
            daySquare.innerText = i;

            const dayString = `${year}-${String(month + 1).padStart(2, '0')}-${String(i).padStart(2, '0')}`;
            
            // Marcar día actual
            const today = new Date();
            if (i === today.getDate() && nav === 0) {
                daySquare.classList.add('today');
            }

            // Marcar evento
            if (events[dayString]) {
                daySquare.classList.add('has-event');
                const dot = document.createElement('div');
                dot.classList.add('event-dot');
                daySquare.appendChild(dot);
                
                // Tooltip simple
                daySquare.title = events[dayString];
            }

            daySquare.addEventListener('click', () => openModal(dayString));
            calendar.appendChild(daySquare);
        }

        renderCountdown();
    }

    function openModal(date) {
        // Uso prompt para mantenerlo simple y funcional sin mucho código extra
        const currentEvent = events[date];
        let msg = `📅 Fecha: ${date}\n`;
        if (currentEvent) msg += `Evento actual: "${currentEvent}"\n\n¿Quieres cambiarlo o borrarlo? (Deja vacío para borrar)`;
        else msg += `Escribe el nombre del Certamen o Entrega:`;

        const newEvent = prompt(msg, currentEvent || '');

        if (newEvent === null) return; // Cancelar

        if (newEvent === '') {
            delete events[date];
        } else {
            events[date] = newEvent;
        }

        localStorage.setItem('misEventos', JSON.stringify(events));
        load();
    }

    function changeMonth(x) {
        nav += x;
        load();
    }

    function renderCountdown() {
        countdownList.innerHTML = '';
        const today = new Date();
        today.setHours(0,0,0,0);

        // Convertir objeto de eventos a array y ordenar por fecha
        const sortedEvents = Object.keys(events)
            .map(date => ({ date: date, title: events[date] }))
            .filter(e => new Date(e.date + 'T00:00:00') >= today) // Solo futuros o hoy
            .sort((a, b) => new Date(a.date) - new Date(b.date));

        if (sortedEvents.length === 0) {
            countdownList.innerHTML = '<p style="color:#777">¡Libre! No hay evaluaciones próximas.</p>';
            return;
        }

        sortedEvents.forEach(e => {
            const eventDate = new Date(e.date + 'T00:00:00');
            const diffTime = eventDate - today;
            const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24)); 

            let colorClass = 'chill';
            let textoDias = `Faltan ${diffDays} días`;

            if (diffDays === 0) {
                textoDias = "¡Es HOY!";
                colorClass = 'urgent';
            } else if (diffDays <= 3) {
                colorClass = 'urgent'; // Rojo
            } else if (diffDays <= 7) {
                colorClass = 'soon'; // Naranja/Amarillo
            }

            const card = document.createElement('div');
            card.className = `event-card ${colorClass}`;
            card.innerHTML = `
                <span class="days-left">${textoDias}</span>
                <div class="event-title">${e.title}</div>
                <div class="event-date">${e.date}</div>
            `;
            countdownList.appendChild(card);
        });
    }

    load();
</script>

</body>
</html>
