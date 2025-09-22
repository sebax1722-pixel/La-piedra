# La-piedra
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>La Piedra del Peñol - Complejo Turístico</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInWithCustomToken, signInAnonymously } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        setLogLevel('Debug');
        
        // Variables globales proporcionadas por el entorno
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // --- Inicio de la lógica de la aplicación ---
        let db, auth;
        let userId;

        // Secciones de la página y modal
        const sections = ['hero', 'acerca', 'menu', 'form', 'precios', 'mapa', 'footer'];
        const photoAlbumModal = document.getElementById('photo-album-modal');
        const formSection = document.getElementById('form-section');
        const formElement = document.getElementById('registro-form');
        const userIdDisplay = document.getElementById('user-id-display');

        // Traducciones
        const translations = {
            es: {
                home: "Inicio", prices: "Precios", map: "Mapa", contact: "Contacto",
                heroTitle: "Descubre la Magia de Guatapé",
                heroSubtitle: "Una aventura inolvidable en la cima de La Piedra del Peñol.",
                ctaButton: "Recibe tu álbum de fotos exclusivo",
                aboutTitle: "Un Lugar de Encuentro con la Naturaleza",
                aboutText: "Nuestro complejo turístico te invita a vivir una experiencia única, combinando la majestuosidad de La Piedra del Peñol con la tranquilidad del embalse de Guatapé. Relájate, explora y conecta con la belleza de la región.",
                attractionsTitle: "Explora la Región",
                attractions1Title: "La Piedra del Peñol", attractions1Text: "Escala sus 659 escalones para una vista panorámica de 360 grados. Una maravilla geológica.",
                attractions2Title: "Guatapé", attractions2Text: "Pasea por el pueblo de los zócalos, famoso por sus coloridas casas y calles vibrantes.",
                attractions3Title: "Medellín", attractions3Text: "A solo dos horas, explora la 'Ciudad de la Eterna Primavera' con su cultura, gastronomía y arte.",
                formTitle: "Obtén tu Álbum de Fotos Exclusivo",
                formSubtitle: "Llena el formulario para recibir un álbum digital de las mejores vistas de la región. ¡Totalmente gratis!",
                name: "Nombre", email: "Correo", phone: "Teléfono", country: "País",
                submit: "Obtener Álbum",
                albumTitle: "Tu Álbum de Fotos Exclusivo",
                albumSubtitle: "¡Gracias por registrarte! Disfruta de estas vistas exclusivas de La Piedra y sus alrededores.",
                close: "Cerrar",
                pricesTitle: "Nuestros Precios",
                pricesText: "Descubre nuestros paquetes de alojamiento y actividades. Ofrecemos opciones para todos los gustos, desde escalada y deportes acuáticos hasta cenas románticas y recorridos turísticos.",
                pricesTable: "Precios de Ejemplos",
                pricesHeader1: "Servicio", pricesHeader2: "Precio",
                pricesRow1: "Tour por la Piedra", pricesPrice1: "$25 USD",
                pricesRow2: "Paseo en lancha", pricesPrice2: "$50 USD",
                pricesRow3: "Cena romántica", pricesPrice3: "$80 USD",
                mapTitle: "Mapa de la Región",
                mapText: "Aquí encontrarás un mapa interactivo con los puntos turísticos más importantes, incluyendo rutas, restaurantes y tiendas de artesanías."
            },
            en: {
                home: "Home", prices: "Prices", map: "Map", contact: "Contact",
                heroTitle: "Discover the Magic of Guatapé",
                heroSubtitle: "An unforgettable adventure at the top of The Rock of Guatapé.",
                ctaButton: "Get your exclusive photo album",
                aboutTitle: "A Place to Connect with Nature",
                aboutText: "Our tourist complex invites you to live a unique experience, combining the majesty of The Rock with the tranquility of the Guatapé reservoir. Relax, explore, and connect with the beauty of the region.",
                attractionsTitle: "Explore the Region",
                attractions1Title: "The Rock of Guatapé", attractions1Text: "Climb its 659 steps for a 360-degree panoramic view. A geological wonder.",
                attractions2Title: "Guatapé", attractions2Text: "Walk through the town of the 'zócalos', famous for its colorful houses and vibrant streets.",
                attractions3Title: "Medellín", attractions3Text: "Just two hours away, explore the 'City of Eternal Spring' with its culture, gastronomy, and art.",
                formTitle: "Get Your Exclusive Photo Album",
                formSubtitle: "Fill out the form to receive a digital album of the best views of the region. It's completely free!",
                name: "Name", email: "Email", phone: "Phone", country: "Country",
                submit: "Get Album",
                albumTitle: "Your Exclusive Photo Album",
                albumSubtitle: "Thank you for registering! Enjoy these exclusive views of The Rock and its surroundings.",
                close: "Close",
                pricesTitle: "Our Prices",
                pricesText: "Discover our accommodation and activity packages. We offer options for all tastes, from climbing and water sports to romantic dinners and tourist tours.",
                pricesTable: "Example Prices",
                pricesHeader1: "Service", pricesHeader2: "Price",
                pricesRow1: "Rock Tour", pricesPrice1: "$25 USD",
                pricesRow2: "Boat Ride", pricesPrice2: "$50 USD",
                pricesRow3: "Romantic Dinner", pricesPrice3: "$80 USD",
                mapTitle: "Regional Map",
                mapText: "Here you will find an interactive map with the most important tourist spots, including routes, restaurants, and craft shops."
            },
            de: {
                home: "Startseite", prices: "Preise", map: "Karte", contact: "Kontakt",
                heroTitle: "Entdecken Sie die Magie von Guatapé",
                heroSubtitle: "Ein unvergessliches Abenteuer auf dem Gipfel des Felsens von El Peñol.",
                ctaButton: "Holen Sie sich Ihr exklusives Fotoalbum",
                aboutTitle: "Ein Ort, um sich mit der Natur zu verbinden",
                aboutText: "Unser Tourismuskomplex lädt Sie zu einem einzigartigen Erlebnis ein, das die Majestät des Felsens mit der Ruhe des Guatapé-Stausees verbindet. Entspannen Sie sich, erkunden Sie und verbinden Sie sich mit der Schönheit der Region.",
                attractionsTitle: "Erkunden Sie die Region",
                attractions1Title: "Der Felsen von El Peñol", attractions1Text: "Erklimmen Sie seine 659 Stufen für eine 360-Grad-Panoramaaussicht. Ein geologisches Wunder.",
                attractions2Title: "Guatapé", attractions2Text: "Spazieren Sie durch die Stadt der 'zócalos', berühmt für ihre farbenfrohen Häuser und lebendigen Straßen.",
                attractions3Title: "Medellín", attractions3Text: "Nur zwei Stunden entfernt, erkunden Sie die 'Stadt des ewigen Frühlings' mit ihrer Kultur, Gastronomie und Kunst.",
                formTitle: "Holen Sie sich Ihr exklusives Fotoalbum",
                formSubtitle: "Füllen Sie das Formular aus, um ein digitales Album mit den besten Ansichten der Region zu erhalten. Völlig kostenlos!",
                name: "Name", email: "E-Mail", phone: "Telefon", country: "Land",
                submit: "Album erhalten",
                albumTitle: "Ihr exklusives Fotoalbum",
                albumSubtitle: "Vielen Dank für Ihre Registrierung! Genießen Sie diese exklusiven Ansichten des Felsens und seiner Umgebung.",
                close: "Schließen",
                pricesTitle: "Unsere Preise",
                pricesText: "Entdecken Sie unsere Unterkunfts- und Aktivitätspakete. Wir bieten Optionen für jeden Geschmack, von Klettern und Wassersport bis hin zu romantischen Abendessen und Touren.",
                pricesTable: "Beispielpreise",
                pricesHeader1: "Dienstleistung", pricesHeader2: "Preis",
                pricesRow1: "Felsentour", pricesPrice1: "25 USD",
                pricesRow2: "Bootsfahrt", pricesPrice2: "50 USD",
                pricesRow3: "Romantisches Abendessen", pricesPrice3: "80 USD",
                mapTitle: "Regionale Karte",
                mapText: "Hier finden Sie eine interaktive Karte mit den wichtigsten Sehenswürdigkeiten, einschließlich Routen, Restaurants und Kunsthandwerksläden."
            },
            fr: {
                home: "Accueil", prices: "Prix", map: "Carte", contact: "Contact",
                heroTitle: "Découvrez la Magie de Guatapé",
                heroSubtitle: "Une aventure inoubliable au sommet de la Pierre du Peñol.",
                ctaButton: "Obtenez votre album photo exclusif",
                aboutTitle: "Un Lieu pour se Connecter à la Nature",
                aboutText: "Notre complexe touristique vous invite à vivre une expérience unique, combinant la majesté de la Pierre avec la tranquillité du réservoir de Guatapé. Détendez-vous, explorez et connectez-vous à la beauté de la région.",
                attractionsTitle: "Explorez la Région",
                attractions1Title: "La Pierre du Peñol", attractions1Text: "Montez ses 659 marches pour une vue panoramique à 360 degrés. Une merveille géologique.",
                attractions2Title: "Guatapé", attractions2Text: "Promenez-vous dans le village des 'zócalos', célèbre pour ses maisons colorées et ses rues animées.",
                attractions3Title: "Medellín", attractions3Text: "À seulement deux heures de là, explorez la 'Ville de l'Éternel Printemps' avec sa culture, sa gastronomie et son art.",
                formTitle: "Obtenez votre Album Photo Exclusif",
                formSubtitle: "Remplissez le formulaire pour recevoir un album numérique des meilleures vues de la région. C'est entièrement gratuit !",
                name: "Nom", email: "Courriel", phone: "Téléphone", country: "Pays",
                submit: "Obtenir l'Album",
                albumTitle: "Votre Album Photo Exclusif",
                albumSubtitle: "Merci de vous être inscrit ! Profitez de ces vues exclusives de la Pierre et de ses environs.",
                close: "Fermer",
                pricesTitle: "Nos Prix",
                pricesText: "Découvrez nos forfaits d'hébergement et d'activités. Nous proposons des options pour tous les goûts, de l'escalade et des sports nautiques aux dîners romantiques et aux visites touristiques.",
                pricesTable: "Exemples de prix",
                pricesHeader1: "Service", pricesHeader2: "Prix",
                pricesRow1: "Tour de la Pierre", pricesPrice1: "25 USD",
                pricesRow2: "Promenade en bateau", pricesPrice2: "50 USD",
                pricesRow3: "Dîner romantique", pricesPrice3: "80 USD",
                mapTitle: "Carte Régionale",
                mapText: "Vous trouverez ici une carte interactive avec les points touristiques les plus importants, y compris les itinéraires, les restaurants et les boutiques d'artisanat."
            }
        };

        let currentLang = 'es';

        // Función para cambiar el idioma de la página
        function setLanguage(lang) {
            currentLang = lang;
            document.documentElement.lang = lang;
            const t = translations[lang];
            document.getElementById('nav-home').textContent = t.home;
            document.getElementById('nav-prices').textContent = t.prices;
            document.getElementById('nav-map').textContent = t.map;
            document.getElementById('nav-contact').textContent = t.contact;
            document.getElementById('hero-title').textContent = t.heroTitle;
            document.getElementById('hero-subtitle').textContent = t.heroSubtitle;
            document.getElementById('cta-button').textContent = t.ctaButton;
            document.getElementById('about-title').textContent = t.aboutTitle;
            document.getElementById('about-text').textContent = t.aboutText;
            document.getElementById('attractions-title').textContent = t.attractionsTitle;
            document.getElementById('attractions-1-title').textContent = t.attractions1Title;
            document.getElementById('attractions-1-text').textContent = t.attractions1Text;
            document.getElementById('attractions-2-title').textContent = t.attractions2Title;
            document.getElementById('attractions-2-text').textContent = t.attractions2Text;
            document.getElementById('attractions-3-title').textContent = t.attractions3Title;
            document.getElementById('attractions-3-text').textContent = t.attractions3Text;
            document.getElementById('form-title').textContent = t.formTitle;
            document.getElementById('form-subtitle').textContent = t.formSubtitle;
            document.getElementById('input-name').placeholder = t.name;
            document.getElementById('input-email').placeholder = t.email;
            document.getElementById('input-phone').placeholder = t.phone;
            document.getElementById('input-country').placeholder = t.country;
            document.getElementById('submit-button').textContent = t.submit;
            document.getElementById('album-title').textContent = t.albumTitle;
            document.getElementById('album-subtitle').textContent = t.albumSubtitle;
            document.getElementById('close-modal').textContent = t.close;
            document.getElementById('prices-title').textContent = t.pricesTitle;
            document.getElementById('prices-text').textContent = t.pricesText;
            document.getElementById('prices-table-title').textContent = t.pricesTable;
            document.getElementById('prices-header-1').textContent = t.pricesHeader1;
            document.getElementById('prices-header-2').textContent = t.pricesHeader2;
            document.getElementById('prices-row-1').textContent = t.pricesRow1;
            document.getElementById('prices-price-1').textContent = t.pricesPrice1;
            document.getElementById('prices-row-2').textContent = t.pricesRow2;
            document.getElementById('prices-price-2').textContent = t.pricesPrice2;
            document.getElementById('prices-row-3').textContent = t.pricesRow3;
            document.getElementById('prices-price-3').textContent = t.pricesPrice3;
            document.getElementById('map-title').textContent = t.mapTitle;
            document.getElementById('map-text').textContent = t.mapText;
        }

        // Función para manejar el scroll del menú
        function handleNavClick(sectionId) {
            document.getElementById(sectionId).scrollIntoView({ behavior: 'smooth' });
        }

        // Función para mostrar el modal del álbum de fotos
        function showPhotoAlbumModal() {
            photoAlbumModal.classList.remove('hidden');
        }

        // Función para cerrar el modal
        function hidePhotoAlbumModal() {
            photoAlbumModal.classList.add('hidden');
        }

        // Función para mostrar un mensaje en un modal
        function showMessage(message) {
            const messageModal = document.getElementById('message-modal');
            const messageText = document.getElementById('message-text');
            messageText.textContent = message;
            messageModal.classList.remove('hidden');
            document.getElementById('close-message-modal').onclick = () => messageModal.classList.add('hidden');
        }

        // Escucha los eventos del DOM
        document.addEventListener('DOMContentLoaded', () => {
            // Inicializa Firebase y autentica
            if (Object.keys(firebaseConfig).length > 0) {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                
                // Autentica de forma anónima o con el token proporcionado
                if (initialAuthToken) {
                    signInWithCustomToken(auth, initialAuthToken).catch(err => {
                        console.error("Error signing in with custom token:", err);
                        signInAnonymously(auth).catch(console.error);
                    });
                } else {
                    signInAnonymously(auth).catch(console.error);
                }

                auth.onAuthStateChanged(user => {
                    if (user) {
                        userId = user.uid;
                        userIdDisplay.textContent = `ID de Usuario: ${userId}`;
                        console.log("User authenticated with UID:", userId);
                        // Escucha en tiempo real para verificar que la DB está funcionando
                        const userRef = collection(db, `/artifacts/${appId}/users/${userId}/contactos`);
                        onSnapshot(userRef, (snapshot) => {
                            console.log("Real-time data update:", snapshot.docs.map(doc => doc.data()));
                        });
                    } else {
                        console.log("No user authenticated.");
                    }
                });

            } else {
                console.warn("Firebase config is missing. Firestore functionality will be disabled.");
            }

            // Maneja el envío del formulario
            formElement.addEventListener('submit', async (e) => {
                e.preventDefault();
                
                if (!db || !userId) {
                    showMessage("Error: La base de datos no está inicializada. Por favor, recarga la página.");
                    return;
                }

                const formData = new FormData(formElement);
                const data = {
                    nombre: formData.get('nombre'),
                    correo: formData.get('correo'),
                    telefono: formData.get('telefono'),
                    pais: formData.get('pais'),
                    fechaRegistro: new Date()
                };

                try {
                    const collectionPath = `/artifacts/${appId}/users/${userId}/contactos`;
                    await addDoc(collection(db, collectionPath), data);
                    console.log("Document successfully written!");
                    formSection.classList.add('hidden');
                    showPhotoAlbumModal();
                } catch (error) {
                    console.error("Error adding document:", error);
                    showMessage("Hubo un error al enviar tus datos. Por favor, inténtalo de nuevo.");
                }
            });

            // Asigna los eventos a los botones y enlaces
            document.getElementById('close-modal').onclick = hidePhotoAlbumModal;
            document.getElementById('cta-button').onclick = () => handleNavClick('form-section');
            document.getElementById('nav-home').onclick = () => handleNavClick('hero-section');
            document.getElementById('nav-prices').onclick = () => handleNavClick('precios-section');
            document.getElementById('nav-map').onclick = () => handleNavClick('mapa-section');
            document.getElementById('nav-contact').onclick = () => handleNavClick('form-section');

            // Establece el idioma por defecto y escucha los cambios
            setLanguage(currentLang);
            document.getElementById('lang-es').onclick = () => setLanguage('es');
            document.getElementById('lang-en').onclick = () => setLanguage('en');
            document.getElementById('lang-de').onclick = () => setLanguage('de');
            document.getElementById('lang-fr').onclick = () => setLanguage('fr');
        });
    </script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;600;700&family=Playfair+Display:wght@700&display=swap');
        
        body {
            font-family: 'Montserrat', sans-serif;
            background-color: #f0fdf4; /* Verde muy claro */
            color: #1f2937;
        }

        .header-title {
            font-family: 'Playfair Display', serif;
        }

        .section {
            padding-top: 5rem;
            padding-bottom: 5rem;
        }

        /* Estilo para el logo SVG */
        .logo-svg {
            pointer-events: none; /* Evita que el usuario haga clic derecho y guarde la imagen */
            user-select: none; /* Evita la selección de texto */
        }
        
        /* Evita que las imágenes sean arrastrables o clicables para guardar */
        img {
            pointer-events: none;
            user-select: none;
            -webkit-user-drag: none;
        }

        /* Estilo para los botones de idioma */
        .lang-button {
            transition: transform 0.2s;
        }
        .lang-button:hover {
            transform: scale(1.1);
        }
    </style>
</head>
<body class="overflow-x-hidden">

    <!-- Mensaje de usuario autenticado -->
    <div id="user-id-display" class="fixed top-2 right-2 z-50 text-xs text-gray-400"></div>

    <!-- Navegación -->
    <nav class="bg-white bg-opacity-90 backdrop-blur-sm fixed top-0 left-0 right-0 z-40 shadow-lg rounded-b-xl">
        <div class="container mx-auto px-6 py-4 flex justify-between items-center">
            <!-- Logo SVG -->
            <div class="flex items-center space-x-2">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="logo-svg w-10 h-10 text-emerald-600">
                    <path fill-rule="evenodd" d="M12 2.25c-5.385 0-9.75 4.365-9.75 9.75s4.365 9.75 9.75 9.75 9.75-4.365 9.75-9.75S17.385 2.25 12 2.25Zm-2.625 6c-.54 0-.975.435-.975.975v.9c0 .54.435.975.975.975h.975a.975.975 0 0 0 .975-.975v-.9a.975.975 0 0 0-.975-.975h-.975Zm4.875 0c-.54 0-.975.435-.975.975v.9c0 .54.435.975.975.975h.975a.975.975 0 0 0 .975-.975v-.9a.975.975 0 0 0-.975-.975h-.975ZM9.75 14.25a.375.375 0 1 0 0 .75h.007a.375.375 0 0 0 0-.75H9.75Zm4.5 0a.375.375 0 1 0 0 .75h.007a.375.375 0 0 0 0-.75H14.25Zm-.75 4.5a.375.375 0 1 0 0 .75h.007a.375.375 0 0 0 0-.75H13.5Zm-4.5 0a.375.375 0 1 0 0 .75h.007a.375.375 0 0 0 0-.75H9Z" clip-rule="evenodd" />
                </svg>
                <span class="header-title text-2xl font-bold text-emerald-800">Peñol Oasis</span>
            </div>
            
            <div class="flex items-center space-x-4">
                <a id="nav-home" href="#hero-section" class="text-gray-600 hover:text-emerald-700 transition duration-300">Inicio</a>
                <a id="nav-prices" href="#precios-section" class="text-gray-600 hover:text-emerald-700 transition duration-300">Precios</a>
                <a id="nav-map" href="#mapa-section" class="text-gray-600 hover:text-emerald-700 transition duration-300">Mapa</a>
                <a id="nav-contact" href="#form-section" class="bg-emerald-500 text-white font-semibold py-2 px-4 rounded-full shadow-lg hover:bg-emerald-600 transition duration-300">Contacto</a>
                
                <!-- Botones de idioma -->
                <button id="lang-es" class="lang-button text-sm text-gray-500 hover:text-emerald-600 font-bold">ES</button>
                <button id="lang-en" class="lang-button text-sm text-gray-500 hover:text-emerald-600 font-bold">EN</button>
                <button id="lang-de" class="lang-button text-sm text-gray-500 hover:text-emerald-600 font-bold">DE</button>
                <button id="lang-fr" class="lang-button text-sm text-gray-500 hover:text-emerald-600 font-bold">FR</button>
            </div>
        </div>
    </nav>

    <!-- Sección Hero (la de bienvenida) -->
    <header id="hero-section" class="section pt-32 bg-gradient-to-br from-green-50 to-teal-100 flex items-center justify-center text-center px-4 md:px-8 h-screen rounded-b-3xl shadow-xl">
        <div class="flex flex-col items-center">
            <h1 id="hero-title" class="header-title text-5xl md:text-7xl font-bold text-gray-800 mb-4 drop-shadow-lg">Descubre la Magia de Guatapé</h1>
            <p id="hero-subtitle" class="text-xl md:text-2xl text-gray-600 mb-8 max-w-3xl">Una aventura inolvidable en la cima de La Piedra del Peñol.</p>
            <button id="cta-button" class="bg-emerald-600 text-white font-bold py-4 px-10 rounded-full shadow-xl hover:bg-emerald-700 transition duration-300 transform hover:scale-105">
                Recibe tu álbum de fotos exclusivo
            </button>
        </div>
    </header>

    <!-- Sección Sobre Nosotros/Atracciones -->
    <main class="container mx-auto">
        <section id="acerca-section" class="section px-4 md:px-8">
            <h2 id="about-title" class="header-title text-4xl text-center font-bold mb-6 text-gray-800">Un Lugar de Encuentro con la Naturaleza</h2>
            <p id="about-text" class="text-center max-w-3xl mx-auto text-lg text-gray-600">
                Nuestro complejo turístico te invita a vivir una experiencia única, combinando la majestuosidad de La Piedra del Peñol con la tranquilidad del embalse de Guatapé. Relájate, explora y conecta con la belleza de la región.
            </p>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-8 mt-12">
                <!-- Imagen y texto 1 -->
                <div class="rounded-2xl overflow-hidden shadow-lg transform transition-all hover:scale-105 duration-300">
                    <img src="https://placehold.co/600x400/9ca3af/ffffff?text=La+Piedra+del+Peñol" alt="La Piedra del Peñol, una gran roca de granito en Guatapé." class="w-full h-48 object-cover object-center">
                    <div class="p-6 bg-white">
                        <h3 id="attractions-1-title" class="font-bold text-xl mb-2 text-emerald-800">La Piedra del Peñol</h3>
                        <p id="attractions-1-text" class="text-gray-700">Escala sus 659 escalones para una vista panorámica de 360 grados. Una maravilla geológica.</p>
                    </div>
                </div>
                <!-- Imagen y texto 2 -->
                <div class="rounded-2xl overflow-hidden shadow-lg transform transition-all hover:scale-105 duration-300">
                    <img src="https://placehold.co/600x400/a3e635/ffffff?text=Pueblo+de+Guatapé" alt="Las coloridas casas del pueblo de Guatapé." class="w-full h-48 object-cover object-center">
                    <div class="p-6 bg-white">
                        <h3 id="attractions-2-title" class="font-bold text-xl mb-2 text-emerald-800">Guatapé</h3>
                        <p id="attractions-2-text" class="text-gray-700">Pasea por el pueblo de los zócalos, famoso por sus coloridas casas y calles vibrantes.</p>
                    </div>
                </div>
                <!-- Imagen y texto 3 -->
                <div class="rounded-2xl overflow-hidden shadow-lg transform transition-all hover:scale-105 duration-300">
                    <img src="https://placehold.co/600x400/34d399/ffffff?text=Medellín" alt="Un paisaje de la ciudad de Medellín." class="w-full h-48 object-cover object-center">
                    <div class="p-6 bg-white">
                        <h3 id="attractions-3-title" class="font-bold text-xl mb-2 text-emerald-800">Medellín</h3>
                        <p id="attractions-3-text" class="text-gray-700">A solo dos horas, explora la 'Ciudad de la Eterna Primavera' con su cultura, gastronomía y arte.</p>
                    </div>
                </div>
            </div>
        </section>

        <!-- Sección de formulario (CTA) -->
        <section id="form-section" class="section bg-emerald-50 rounded-3xl shadow-lg my-12 mx-4 md:mx-8">
            <div class="max-w-xl mx-auto p-8 text-center">
                <h2 id="form-title" class="header-title text-4xl font-bold mb-4 text-emerald-800">Obtén tu Álbum de Fotos Exclusivo</h2>
                <p id="form-subtitle" class="text-gray-600 mb-8">
                    Llena el formulario para recibir un álbum digital de las mejores vistas de la región. ¡Totalmente gratis!
                </p>
                <form id="registro-form" class="space-y-4">
                    <input id="input-name" type="text" name="nombre" placeholder="Nombre" required class="w-full p-3 rounded-lg border border-gray-300 focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 transition duration-200">
                    <input id="input-email" type="email" name="correo" placeholder="Correo" required class="w-full p-3 rounded-lg border border-gray-300 focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 transition duration-200">
                    <input id="input-phone" type="tel" name="telefono" placeholder="Teléfono" class="w-full p-3 rounded-lg border border-gray-300 focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 transition duration-200">
                    <input id="input-country" type="text" name="pais" placeholder="País" class="w-full p-3 rounded-lg border border-gray-300 focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 transition duration-200">
                    <button id="submit-button" type="submit" class="w-full bg-emerald-600 text-white font-bold py-3 rounded-lg shadow-md hover:bg-emerald-700 transition duration-300">
                        Obtener Álbum
                    </button>
                </form>
            </div>
        </section>
        
        <!-- Sección de Precios -->
        <section id="precios-section" class="section px-4 md:px-8">
            <h2 id="prices-title" class="header-title text-4xl text-center font-bold mb-6 text-gray-800">Nuestros Precios</h2>
            <p id="prices-text" class="text-center max-w-3xl mx-auto text-lg text-gray-600">
                Descubre nuestros paquetes de alojamiento y actividades. Ofrecemos opciones para todos los gustos, desde escalada y deportes acuáticos hasta cenas románticas y recorridos turísticos.
            </p>
            <div class="max-w-2xl mx-auto mt-12 bg-white rounded-xl shadow-lg p-6">
                <h3 id="prices-table-title" class="font-bold text-2xl mb-4 text-emerald-800">Precios de Ejemplos</h3>
                <div class="overflow-x-auto">
                    <table class="w-full text-left table-auto">
                        <thead>
                            <tr class="border-b-2 border-emerald-200">
                                <th id="prices-header-1" class="py-3 px-4 font-semibold text-gray-700">Servicio</th>
                                <th id="prices-header-2" class="py-3 px-4 font-semibold text-gray-700">Precio</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr class="border-b border-gray-200">
                                <td id="prices-row-1" class="py-3 px-4 text-gray-600">Tour por la Piedra</td>
                                <td id="prices-price-1" class="py-3 px-4 font-semibold text-emerald-600">$25 USD</td>
                            </tr>
                            <tr class="border-b border-gray-200">
                                <td id="prices-row-2" class="py-3 px-4 text-gray-600">Paseo en lancha</td>
                                <td id="prices-price-2" class="py-3 px-4 font-semibold text-emerald-600">$50 USD</td>
                            </tr>
                            <tr>
                                <td id="prices-row-3" class="py-3 px-4 text-gray-600">Cena romántica</td>
                                <td id="prices-price-3" class="py-3 px-4 font-semibold text-emerald-600">$80 USD</td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- Sección de Mapa -->
        <section id="mapa-section" class="section px-4 md:px-8">
            <h2 id="map-title" class="header-title text-4xl text-center font-bold mb-6 text-gray-800">Mapa de la Región</h2>
            <p id="map-text" class="text-center max-w-3xl mx-auto text-lg text-gray-600">
                Aquí encontrarás un mapa interactivo con los puntos turísticos más importantes, incluyendo rutas, restaurantes y tiendas de artesanías.
            </p>
            <div class="mt-12 w-full max-w-5xl mx-auto bg-white rounded-xl shadow-lg overflow-hidden">
                <iframe 
                    src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d15858.989808386307!2d-75.17646695!3d6.2166946!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x8e420b7012674e79%3A0x6a2c270d1822847c!2sPiedra%20del%20Pe%C3%B1ol!5e0!3m2!1ses-419!2sco!4v1717361734208!5m2!1ses-419!2sco" 
                    width="100%" 
                    height="450" 
                    style="border:0;" 
                    allowfullscreen="" 
                    loading="lazy" 
                    referrerpolicy="no-referrer-when-downgrade">
                </iframe>
            </div>
        </section>
    </main>

    <!-- Footer -->
    <footer id="footer" class="bg-gray-800 text-gray-300 text-center p-8 mt-12 rounded-t-3xl shadow-inner">
        <p>&copy; 2024 Complejo Turístico La Piedra del Peñol. Todos los derechos reservados.</p>
    </footer>

    <!-- Modal para el Álbum de Fotos -->
    <div id="photo-album-modal" class="fixed inset-0 bg-black bg-opacity-70 flex items-center justify-center z-50 p-4 hidden">
        <div class="bg-white rounded-2xl shadow-xl p-8 max-w-3xl w-full text-center relative transform scale-95 transition-all duration-300">
            <h3 id="album-title" class="header-title text-3xl font-bold text-emerald-800 mb-2">Tu Álbum de Fotos Exclusivo</h3>
            <p id="album-subtitle" class="text-gray-600 mb-6">¡Gracias por registrarte! Disfruta de estas vistas exclusivas de La Piedra y sus alrededores.</p>
            
            <!-- Galería de imágenes con estilo que evita guardarlas -->
            <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4">
                <img src="https://placehold.co/600x400/1e293b/ffffff?text=Guatapé+Panorámica" alt="Panorámica del embalse de Guatapé desde la cima de la Piedra del Peñol." class="rounded-xl w-full h-48 object-cover">
                <img src="https://placehold.co/600x400/a3e635/ffffff?text=Pueblo+y+Embalce" alt="Vista aérea del pueblo de Guatapé y el embalse." class="rounded-xl w-full h-48 object-cover">
                <img src="https://placehold.co/600x400/34d399/ffffff?text=Paisaje+Verde" alt="Paisaje verde de la zona de Antioquia." class="rounded-xl w-full h-48 object-cover">
                <img src="https://placehold.co/600x400/9ca3af/ffffff?text=Atardecer+en+la+Piedra" alt="Atardecer sobre el embalse de Guatapé." class="rounded-xl w-full h-48 object-cover">
                <img src="https://placehold.co/600x400/1e293b/ffffff?text=Cascada+Cercana" alt="Vista de una cascada en la región cercana a Guatapé." class="rounded-xl w-full h-48 object-cover">
                <img src="https://placehold.co/600x400/a3e635/ffffff?text=Vuelo+en+Globo" alt="Vuelo en globo sobre la Piedra del Peñol y Guatapé." class="rounded-xl w-full h-48 object-cover">
            </div>

            <button id="close-modal" class="mt-8 bg-gray-200 text-gray-700 font-semibold py-2 px-6 rounded-full hover:bg-gray-300 transition duration-300">
                Cerrar
            </button>
        </div>
    </div>

    <!-- Modal de Mensajes (Alert) -->
    <div id="message-modal" class="fixed inset-0 bg-black bg-opacity-70 flex items-center justify-center z-50 p-4 hidden">
        <div class="bg-white rounded-xl shadow-2xl p-6 max-w-sm w-full text-center">
            <p id="message-text" class="text-lg text-gray-800 mb-4"></p>
            <button id="close-message-modal" class="bg-emerald-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-emerald-700 transition">Cerrar</button>
        </div>
    </div>

</body>
</html>
