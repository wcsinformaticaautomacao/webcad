<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel ADM | Montesanto Semijoias</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        gold: { 500: '#D4AF37', 600: '#AA8C2C' },
                        montesanto: { black: '#1A1A1A', dark: '#2C2C2C' }
                    }
                }
            }
        }
    </script>
</head>
<body class="bg-gray-100 min-h-screen font-sans">

    <!-- Tela de Login -->
    <div id="loginPage" class="flex items-center justify-center min-h-screen p-4">
        <div class="bg-white p-8 rounded-lg shadow-2xl w-full max-w-md border-t-4 border-gold-500">
            <div class="text-center mb-8">
                <h1 class="text-3xl font-serif font-bold text-montesanto-black">ADMINISTRAÇÃO</h1>
                <p class="text-gold-600 text-sm tracking-widest uppercase">Montesanto Semijoias</p>
            </div>
            <form id="loginForm" class="space-y-6">
                <div>
                    <label class="block text-sm font-medium text-gray-700">E-mail</label>
                    <input type="email" id="loginEmail" required class="w-full px-4 py-2 border rounded-md focus:ring-gold-500 outline-none">
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700">Senha</label>
                    <input type="password" id="loginPassword" required class="w-full px-4 py-2 border rounded-md focus:ring-gold-500 outline-none">
                </div>
                <button type="submit" class="w-full bg-montesanto-black text-gold-500 font-bold py-3 rounded-md hover:bg-gold-600 hover:text-white transition">
                    Entrar no Sistema
                </button>
            </form>
        </div>
    </div>

    <!-- Painel de Controle (Dashboard) -->
    <div id="adminDashboard" class="hidden">
        <nav class="bg-montesanto-black text-white p-4 shadow-md">
            <div class="container mx-auto flex justify-between items-center">
                <span class="font-serif text-xl text-gold-500">Montesanto ADM</span>
                <button onclick="logout()" class="text-sm text-gray-300 hover:text-white"><i class="fas fa-sign-out-alt mr-2"></i>Sair</button>
            </div>
        </nav>

        <div class="container mx-auto p-4 md:p-8">
            <div class="flex flex-col md:flex-row justify-between items-start md:items-center mb-8 gap-4">
                <h2 class="text-2xl font-bold text-gray-800">Gerenciar Revendedoras</h2>
                <div class="flex gap-2">
                    <span class="bg-yellow-100 text-yellow-800 px-3 py-1 rounded-full text-xs font-bold uppercase">Pendentes</span>
                    <span class="bg-green-100 text-green-800 px-3 py-1 rounded-full text-xs font-bold uppercase">Aprovadas</span>
                </div>
            </div>

            <!-- Tabela de Cadastros -->
            <div class="bg-white rounded-lg shadow-md overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-gray-50 border-b">
                            <th class="p-4 text-xs font-bold text-gray-500 uppercase">Data</th>
                            <th class="p-4 text-xs font-bold text-gray-500 uppercase">Nome / WhatsApp</th>
                            <th class="p-4 text-xs font-bold text-gray-500 uppercase">Localização</th>
                            <th class="p-4 text-xs font-bold text-gray-500 uppercase">Documentos</th>
                            <th class="p-4 text-xs font-bold text-gray-500 uppercase">Status</th>
                            <th class="p-4 text-xs font-bold text-gray-500 uppercase text-center">Ações</th>
                        </tr>
                    </thead>
                    <tbody id="resellerList">
                        <!-- Dados carregados via JavaScript -->
                        <tr>
                            <td colspan="6" class="p-10 text-center text-gray-400">Carregando cadastros...</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-app.js";
        import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-auth.js";
        import { getFirestore, collection, onSnapshot, doc, updateDoc, query, orderBy } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-firestore.js";

        // CONFIGURAÇÃO DO FIREBASE (COLE A MESMA DO SEU ARQUIVO DE CADASTRO)
        const firebaseConfig = {
            apiKey: "COLE_SUA_API_KEY_AQUI",
            authDomain: "seu-projeto.firebaseapp.com",
            projectId: "seu-projeto",
            storageBucket: "seu-projeto.appspot.com",
            messagingSenderId: "000000000",
            appId: "seu-app-id"
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        // LOGIN
        document.getElementById('loginForm').onsubmit = async (e) => {
            e.preventDefault();
            const email = document.getElementById('loginEmail').value;
            const pass = document.getElementById('loginPassword').value;
            try {
                await signInWithEmailAndPassword(auth, email, pass);
            } catch (error) {
                alert("Acesso Negado: E-mail ou senha inválidos.");
            }
        };

        window.logout = () => signOut(auth);

        // MONITOR DE AUTENTICAÇÃO
        onAuthStateChanged(auth, (user) => {
            if (user) {
                document.getElementById('loginPage').classList.add('hidden');
                document.getElementById('adminDashboard').classList.remove('hidden');
                loadResellers();
            } else {
                document.getElementById('loginPage').classList.remove('hidden');
                document.getElementById('adminDashboard').classList.add('hidden');
            }
        });

        // CARREGAR DADOS
        function loadResellers() {
            const q = query(collection(db, "revendedoras"), orderBy("data_cadastro", "desc"));
            
            onSnapshot(q, (snapshot) => {
                const list = document.getElementById('resellerList');
                list.innerHTML = "";
                
                snapshot.forEach((docSnap) => {
                    const data = docSnap.data();
                    const id = docSnap.id;
                    const date = new Date(data.data_cadastro).toLocaleDateString('pt-BR');
                    
                    const statusColors = {
                        pendente: 'bg-yellow-100 text-yellow-700',
                        aprovada: 'bg-green-100 text-green-700',
                        recusada: 'bg-red-100 text-red-700'
                    };

                    list.innerHTML += `
                        <tr class="border-b hover:bg-gray-50 transition">
                            <td class="p-4 text-sm text-gray-500">${date}</td>
                            <td class="p-4">
                                <div class="font-bold text-gray-800">${data.nome}</div>
                                <div class="text-xs text-green-600"><i class="fab fa-whatsapp"></i> ${data.whatsapp}</div>
                            </td>
                            <td class="p-4 text-xs text-gray-600">
                                ${data.cidade} - ${data.estado}<br>${data.bairro}
                            </td>
                            <td class="p-4">
                                <div class="flex flex-wrap gap-1">
                                    <a href="${data.documentos.rg_frente}" target="_blank" class="px-2 py-1 bg-gray-100 text-[10px] rounded hover:bg-gold-500 hover:text-white">RG FRENTE</a>
                                    <a href="${data.documentos.rg_verso}" target="_blank" class="px-2 py-1 bg-gray-100 text-[10px] rounded hover:bg-gold-500 hover:text-white">RG VERSO</a>
                                    <a href="${data.documentos.residencia}" target="_blank" class="px-2 py-1 bg-gray-100 text-[10px] rounded hover:bg-gold-500 hover:text-white">RESIDÊNCIA</a>
                                </div>
                            </td>
                            <td class="p-4 text-center">
                                <span class="px-3 py-1 rounded-full text-[10px] font-bold uppercase ${statusColors[data.status] || 'bg-gray-100'}">
                                    ${data.status}
                                </span>
                            </td>
                            <td class="p-4">
                                <div class="flex justify-center gap-2">
                                    <button onclick="updateStatus('${id}', 'aprovada')" class="text-green-500 hover:text-green-700 p-2" title="Aprovar"><i class="fas fa-check-circle text-xl"></i></button>
                                    <button onclick="updateStatus('${id}', 'recusada')" class="text-red-500 hover:text-red-700 p-2" title="Recusar"><i class="fas fa-times-circle text-xl"></i></button>
                                </div>
                            </td>
                        </tr>
                    `;
                });
                
                if (snapshot.empty) {
                    list.innerHTML = '<tr><td colspan="6" class="p-10 text-center text-gray-400">Nenhum cadastro encontrado.</td></tr>';
                }
            });
        }

        // ATUALIZAR STATUS
        window.updateStatus = async (id, novoStatus) => {
            if(confirm(`Deseja alterar o status para ${novoStatus.toUpperCase()}?`)) {
                try {
                    const docRef = doc(db, "revendedoras", id);
                    await updateDoc(docRef, { status: novoStatus });
                } catch (e) {
                    alert("Erro ao atualizar status.");
                }
            }
        };
    </script>
</body>
</html>
