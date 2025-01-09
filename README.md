# UAS
NAMA = TEGAR TAQIF YASSAR 
NIM  = 21.01.55.0014

```php
<?php
session_start();

// Cek apakah user sudah login
if(!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
?>
<!DOCtype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Daftar elektroniks</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body{
            background-color : #D3D3D3;
        }
    </style>
</head>
<body>
<div class="container mt-3 text-center">
    <div class="card">
    <div class="card-body">
<div class="card-title"><h2>Welcome, <?php echo $_SESSION['name']; ?>!</h2></div>
<div class="card-text"><p>username : <?php echo $_SESSION['username']; ?></p></div>
<div ><a href="logout.php"><button type="button" class="col-sm-4 btn btn-outline-secondary" data-bs-dismiss="modal">Logout</button></a></div>
</div>

<div class="container mt-5">
        <h2 class="mb-4">Daftar elektronik</h2>
        
        <!-- Form Pencarian dan Tombol Tambah -->
        <div class="row mb-3">
        <div class="col">
            <input type="text" id="searchInput" class="form-control" placeholder="Cari berdasarkan ID">
        </div>
        <div class="col-auto">
            <button onclick="searchelektronik()" class="btn btn-primary">Cari</button>
        </div>
        <div class="col-auto">
            <button type="button" class="btn btn-success" data-bs-toggle="modal" data-bs-target="#elektronikModal">
                Tambah Buku
            </button>
        </div>
    </div>

        <div class="table-responsive">
            <table class="table table-striped">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>name</th>
                        <th>brand</th>
                        <th>price</th>
                        <th>Stock</th>
                        <th>Aksi</th>
                    </tr>
                </thead>
                <tbody id="elektronikList" >
                    <?php
                    try {
                        // Set URL API
                        $api_url = 'http://localhost/elektronik/elektronik.php';
                        
                        // Cek jika ada pencarian
                        if(isset($_GET['search_id']) && !empty($_GET['search_id'])) {
                            $api_url .= '/' . $_GET['search_id'];
                        }
                        
                        // Konfigurasi timeout
                        $ctx = stream_context_create([
                            'http' => ['timeout' => 5]
                        ]);
                        
                        // Ambil data dari API
                        $response = file_get_contents($api_url, false, $ctx);
                        
                        // Validasi response
                        if ($response === false) {
                            throw new Exception('Gagal mengambil data dari API');
                        }
                        
                        // Validasi JSON
                        $data = json_decode($response, true);
                        if (json_last_error() !== JSON_ERROR_NONE) {
                            throw new Exception('Invalid JSON response');
                        }

                        // Tampilkan data berdasarkan jenis request
                        if(isset($_GET['search_id'])) {
                            if($data && !isset($data['message'])) {
                                // Tampilkan satu buku
                                echo "<tr>";
                                echo "<td>{$data['id']}</td>";
                                echo "<td>{$data['name']}</td>";
                                echo "<td>{$data['brand']}</td>";
                                echo "<td>{$data['price']}</td>";
                                echo "<td>{$data['stock']}</td>";
                                echo "</tr>";
                            } else {
                                // Tampilkan pesan tidak ditemukan
                                echo "<tr><td colspan='4' class='text-center'>";
                                echo "Buku dengan ID tersebut tidak ditemukan";
                                echo "</td></tr>";
                            }
                        } else {
                            // Cek apakah ada data
                            if(!empty($data)) {
                                // Loop untuk setiap buku
                                foreach($data as $db) {
                                    echo "<tr>";
                                    echo "<td>{$db['id']}</td>";
                                    echo "<td>{$db['name']}</td>";
                                    echo "<td>{$db['brand']}</td>";
                                    echo "<td>{$db['price']}</td>";
                                    echo "<td>{$db['stock']}</td>";
                                    echo "</tr>";
                                }
                            } else {
                                // Tampilkan pesan jika tidak ada data
                                echo "<tr><td colspan='4' class='text-center'>";
                                echo "Tidak ada data buku";
                                echo "</td></tr>";
                            }
                        }
                        
                    } catch (Exception $e) {
                        echo "<tr><td colspan='4' class='text-center text-danger'>";
                        echo "Error: " . $e->getMessage();
                        echo "</td></tr>";
                    }
                    ?>
                </tbody>
            </table>
        </div>
    </div>

    <!-- Modal Tambah Buku -->
    <div class="modal fade" id="adddbModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-name">Tambah elektronik Baru</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                    <form id="adddbForm">
                        <div class="mb-3">
                            <label class="form-label">name</label>
                            <input type="text" name="name" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">brand</label>
                            <input type="text" name="brand" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">price</label>
                            <input type="text" name="price" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Stock</label>
                            <input type="number" name="stock" class="form-control" required>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                    <button type="button" class="btn btn-primary" onclick="adddb()">Simpan</button>
                </div>
            </div>
        </div>
    </div>
    <!-- Modal for Add/Edit elektronik -->
    <div class="modal fade" id="elektronikModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-name" id="modalName">Tambah Buku</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                    <form id="elektronikForm">
                        <input type="hidden" id="elektronikId">
                        <div class="mb-3">
                            <label for="name" class="form-label">name</label>
                            <input type="text" class="form-control" id="name" required>
                        </div>
                        <div class="mb-3">
                            <label for="brand" class="form-label">brand</label>
                            <input type="text" class="form-control" id="brand" required>
                        </div>
                        <div class="mb-3">
                            <label for="price" class="form-label">price</label>
                            <input type="text" class="form-control" id="price" required>
                        </div>
                        <div class="mb-3">
                            <label for="stock" class="form-label">Stock</label>
                            <input type="number" class="form-control" id="stock" required>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                    <button type="button" class="btn btn-primary" onclick="saveelektronik()">Simpan</button>
                </div>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        const API_URL = 'http://localhost/elektronik/elektronik.php';
        let elektronikModal;

        document.addEventListener('DOMContentLoaded', function() {
            elektronikModal = new bootstrap.Modal(document.getElementById('elektronikModal'));
            loadelektroniks();
        });

        function loadelektroniks() {
            fetch(API_URL)
                .then(response => response.json())
                .then(elektroniks => {
                    const elektronikList = document.getElementById('elektronikList');
                    elektronikList.innerHTML = '';
                    elektroniks.forEach(elektronik => {
                        elektronikList.innerHTML += `
                            <tr>
                                <td>${elektronik.id}</td>
                                <td>${elektronik.name}</td>
                                <td>${elektronik.brand}</td>
                                <td>${elektronik.price}</td>
                                <td>${elektronik.stock}</td>
                                <td class="btn-group-action">
                                    <button class="btn btn-sm btn-warning me-1" onclick="editelektronik(${elektronik.id})">Edit</button>
                                    <button class="btn btn-sm btn-danger" onclick="deleteelektronik(${elektronik.id})">Hapus</button>
                                </td>
                            </tr>
                        `;
                    });
                })
                .catch(error => alert('Error loading elektroniks: ' + error));
        }

        function searchelektronik() {
            const id = document.getElementById('searchInput').value;
            if (!id) {
                loadelektroniks();
                return;
            }
            
            fetch(`${API_URL}/${id}`)
                .then(response => response.json())
                .then(elektronik => {
                    const elektronikList = document.getElementById('elektronikList');
                    if (elektronik.message) {
                        alert('elektronik not found');
                        return;
                    }
                    elektronikList.innerHTML = `
                        <tr>
                            <td>${elektronik.id}</td>
                            <td>${elektronik.name}</td>
                            <td>${elektronik.brand}</td>
                            <td>${elektronik.price}</td>
                            <td>${elektronik.stock}</td>
                            <td class="btn-group-action">
                                <button class="btn btn-sm btn-warning me-1" onclick="editelektronik(${elektronik.id})">Edit</button>
                                <button class="btn btn-sm btn-danger" onclick="deleteelektronik(${elektronik.id})">Hapus</button>
                            </td>
                        </tr>
                    `;
                })
                .catch(error => alert('Error searching elektronik: ' + error));
        }

        function editelektronik(id) {
            fetch(`${API_URL}/${id}`)
                .then(response => response.json())
                .then(elektronik => {
                    document.getElementById('elektronikId').value = elektronik.id;
                    document.getElementById('name').value = elektronik.name;
                    document.getElementById('brand').value = elektronik.brand;
                    document.getElementById('price').value = elektronik.price;
                    document.getElementById('stock').value = elektronik.stock;
                    document.getElementById('modalName').textContent = 'Edit Buku';
                    elektronikModal.show();
                })
                .catch(error => alert('Error loading elektronik details: ' + error));
        }

        function deleteelektronik(id) {
            if (confirm('Are you sure you want to delete this elektronik?')) {
                fetch(`${API_URL}/${id}`, {
                    method: 'DELETE'
                })
                .then(response => response.json())
                .then(data => {
                    alert('elektronik deleted successfully');
                    loadelektroniks();
                })
                .catch(error => alert('Error deleting elektronik: ' + error));
            }
        }

        function saveelektronik() {
            const elektronikId = document.getElementById('elektronikId').value;
            const elektronikData = {
                name: document.getElementById('name').value,
                brand: document.getElementById('brand').value,
                price: document.getElementById('price').value,
                stock: document.getElementById('stock').value
            };

            const method = elektronikId ? 'PUT' : 'POST';
            const url = elektronikId ? `${API_URL}/${elektronikId}` : API_URL;

            fetch(url, {
                method: method,
                headers: {
                    'Content-brand': 'application/json'
                },
                body: JSON.stringify(elektronikData)
            })
            .then(response => response.json())
            .then(data => {
                alert(elektronikId ? 'elektronik updated successfully' : 'elektronik added successfully');
                elektronikModal.hide();
                loadelektroniks();
                resetForm();
            })
            .catch(error => alert('Error saving elektronik: ' + error));
        }

        function resetForm() {
            document.getElementById('elektronikId').value = '';
            document.getElementById('elektronikForm').reset();
            document.getElementById('modalName').textContent = 'Tambah Buku';
        }

        // Reset form when modal is closed
        document.getElementById('elektronikModal').addEventListener('hidden.bs.modal', resetForm);
    </script>
    
    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
