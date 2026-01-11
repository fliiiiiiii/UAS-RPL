# UAS-RPL

# *SEWA KENDARAAN* #
*Rafli Anugrah Ramadhan*
*312410351*

# 1. Config: #
~~~ php
<?php
$conn = mysqli_connect("localhost", "root", "", "sewa_kendaraan");
date_default_timezone_set('Asia/Jakarta');
if (!$conn) die("Koneksi gagal: " . mysqli_connect_error());
?>
~~~

# 2. Dashboard: #
~~~ php
<div class="container mt-4">
    <div class="card shadow-sm p-5 bg-primary text-white text-center">
        <h1>Selamat Datang, <?= $_SESSION['user']; ?>!</h1>
        <p>Aplikasi Rental Mobil - Kelola armada dan transaksi Anda di sini.</p>
    </div>
</div>
~~~

# 3. Hapus: #
~~~ php
<?php
include 'config.php';
session_start();
if(!isset($_SESSION['admin'])) { header("Location: login.php"); exit; }

// Hapus Kendaraan
if(isset($_GET['id_k'])){
    $id = $_GET['id_k'];
    mysqli_query($conn, "DELETE FROM kendaraan WHERE id = '$id'");
    header("Location: kendaraan.php");
}

// Hapus Pelanggan
if(isset($_GET['id_p'])){
    $id = $_GET['id_p'];
    mysqli_query($conn, "DELETE FROM pelanggan WHERE id = '$id'");
    header("Location: pelanggan.php");
}
?>
~~~

# 4. Index: #
~~~ php
<?php 
include 'config.php'; 
session_start();

// Proteksi Halaman: Jika belum login, lempar ke login.php
if(!isset($_SESSION['admin'])) { 
    header("Location: login.php"); 
    exit; 
}

// 1. HITUNG STATISTIK UNTUK WIDGET
// Menghitung total semua armada
$total_unit = mysqli_fetch_assoc(mysqli_query($conn, "SELECT COUNT(*) as t FROM kendaraan"))['t'];
// Menghitung unit yang sedang disewa (Pengelolaan Stok)
$unit_sewa = mysqli_fetch_assoc(mysqli_query($conn, "SELECT COUNT(*) as t FROM kendaraan WHERE status='Disewa'"))['t'];
// Menghitung total uang masuk (Sewa + Denda)
$pendapatan = mysqli_fetch_assoc(mysqli_query($conn, "SELECT SUM(total_bayar + denda) as t FROM transaksi"))['t'];
?>

<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard - SewaApp</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.0/font/bootstrap-icons.css">
    <style>
        body { background-color: #f8f9fa; }
        .sidebar { min-height: 100vh; background: #212529; color: white; padding-top: 20px; }
        .nav-link { color: #adb5bd; margin-bottom: 10px; transition: 0.3s; }
        .nav-link:hover, .nav-link.active { color: white; background: #343a40; border-radius: 5px; }
        .card-stat { border: none; border-radius: 15px; transition: transform 0.3s; }
        .card-stat:hover { transform: translateY(-5px); }
    </style>
</head>
<body>

<div class="container-fluid">
    <div class="row">
        <nav class="col-md-2 d-none d-md-block sidebar shadow">
            <div class="px-3 mb-4">
                <h4 class="fw-bold"><i class="bi bi-car-front-fill me-2"></i>SEWA APP</h4>
                <small class="text-muted">Manajemen Rental</small>
            </div>
            <ul class="nav flex-column px-2">
                <li class="nav-item"><a class="nav-link active" href="index.php"><i class="bi bi-speedometer2 me-2"></i> Dashboard</a></li>
                <li class="nav-item"><a class="nav-link" href="kendaraan.php"><i class="bi bi-truck me-2"></i> Master Kendaraan</a></li>
                <li class="nav-item"><a class="nav-link" href="pelanggan.php"><i class="bi bi-people me-2"></i> Master Pelanggan</a></li>
                <li class="nav-item"><a class="nav-link" href="transaksi.php"><i class="bi bi-cart-check me-2"></i> Transaksi Sewa</a></li>
                <li class="nav-item mt-4"><a class="nav-link text-danger" href="logout.php"><i class="bi bi-box-arrow-right me-2"></i> Logout</a></li>
            </ul>
        </nav>

        <main class="col-md-10 ms-sm-auto px-md-4 py-4">
            <div class="d-flex justify-content-between align-items-center mb-4">
                <h2 class="fw-bold">Ringkasan Bisnis</h2>
                <span class="badge bg-dark px-3 py-2"><i class="bi bi-calendar3 me-2"></i> <?= date('d M Y') ?></span>
            </div>

            <div class="row g-3 mb-4">
                <div class="col-md-4">
                    <div class="card card-stat bg-primary text-white shadow-sm">
                        <div class="card-body p-4">
                            <div class="d-flex justify-content-between border-0">
                                <div>
                                    <h6>Total Armada</h6>
                                    <h2 class="fw-bold mb-0"><?= $total_unit ?> <small style="font-size: 15px;">Unit</small></h2>
                                </div>
                                <i class="bi bi-truck" style="font-size: 2.5rem; opacity: 0.5;"></i>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-md-4">
                    <div class="card card-stat bg-warning text-dark shadow-sm">
                        <div class="card-body p-4">
                            <div class="d-flex justify-content-between">
                                <div>
                                    <h6>Sedang Disewa</h6>
                                    <h2 class="fw-bold mb-0"><?= $unit_sewa ?> <small style="font-size: 15px;">Unit</small></h2>
                                </div>
                                <i class="bi bi-key" style="font-size: 2.5rem; opacity: 0.5;"></i>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-md-4">
                    <div class="card card-stat bg-success text-white shadow-sm">
                        <div class="card-body p-4">
                            <div class="d-flex justify-content-between">
                                <div>
                                    <h6>Total Pendapatan</h6>
                                    <h2 class="fw-bold mb-0">Rp <?= number_format($pendapatan) ?></h2>
                                </div>
                                <i class="bi bi-wallet2" style="font-size: 2.5rem; opacity: 0.5;"></i>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="card border-0 shadow-sm">
                <div class="card-header bg-white py-3">
                    <h5 class="mb-0 fw-bold text-dark">Riwayat Transaksi & Laporan</h5>
                </div>
                <div class="card-body p-0">
                    <div class="table-responsive">
                        <table class="table table-hover align-middle mb-0">
                            <thead class="table-light">
                                <tr>
                                    <th class="ps-4">No. Transaksi</th>
                                    <th>Pelanggan</th>
                                    <th>Unit Kendaraan</th>
                                    <th>Tgl Sewa</th>
                                    <th>Total Bayar</th>
                                    <th>Denda</th>
                                    <th>Status</th>
                                    <th class="text-center">Opsi</th>
                                </tr>
                            </thead>
                            <tbody>
                                <?php
                                $query = "SELECT t.*, p.nama as p_nama, k.nama as k_nama, k.jenis as k_jenis 
                                          FROM transaksi t 
                                          JOIN pelanggan p ON t.id_pelanggan = p.id 
                                          JOIN kendaraan k ON t.id_kendaraan = k.id 
                                          ORDER BY t.id DESC";
                                $res = mysqli_query($conn, $query);
                                
                                if(mysqli_num_rows($res) == 0) {
                                    echo "<tr><td colspan='8' class='text-center py-4 text-muted'>Belum ada riwayat transaksi.</td></tr>";
                                }

                                while($row = mysqli_fetch_assoc($res)) {
                                    $status_badge = ($row['status'] == 'Aktif') ? 'bg-warning text-dark' : 'bg-success';
                                    ?>
                                    <tr>
                                        <td class="ps-4 fw-bold">#TRX-<?= $row['id'] ?></td>
                                        <td><?= $row['p_nama'] ?></td>
                                        <td>
                                            <?= $row['k_nama'] ?> 
                                            <br><small class="text-muted"><?= $row['k_jenis'] ?></small>
                                        </td>
                                        <td><?= date('d/m/Y', strtotime($row['tgl_sewa'])) ?></td>
                                        <td>Rp <?= number_format($row['total_bayar']) ?></td>
                                        <td class="text-danger fw-bold">Rp <?= number_format($row['denda']) ?></td>
                                        <td><span class="badge <?= $status_badge ?>"><?= $row['status'] ?></span></td>
                                        <td class="text-center">
                                            <a href="nota.php?id=<?= $row['id'] ?>" target="_blank" class="btn btn-sm btn-outline-dark">
                                                <i class="bi bi-printer me-1"></i> Nota
                                            </a>
                                        </td>
                                    </tr>
                                    <?php
                                }
                                ?>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </main>
    </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
~~~

# 5. Kendaraan: #
~~~ php
<?php 
include 'config.php'; 
session_start();
if(!isset($_SESSION['admin'])) { header("Location: login.php"); exit; }

// PROSES TAMBAH
if(isset($_POST['tambah'])){
    $nama = $_POST['nama']; $jenis = $_POST['jenis']; $plat = $_POST['plat']; $harga = $_POST['harga'];
    mysqli_query($conn, "INSERT INTO kendaraan (nama, jenis, plat_nomor, harga_sewa) VALUES ('$nama', '$jenis', '$plat', '$harga')");
    header("Location: kendaraan.php");
}

// PROSES UPDATE
if(isset($_POST['update'])){
    $id = $_POST['id'];
    $nama = $_POST['nama'];
    $jenis = $_POST['jenis'];
    $harga = $_POST['harga_sewa'];
    $plat = $_POST['plat_nomor'];

    mysqli_query($conn, "UPDATE kendaraan SET nama='$nama', jenis='$jenis', harga_sewa='$harga', plat_nomor='$plat' WHERE id='$id'");
    header("Location: kendaraan.php");
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Master Kendaraan</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light p-4">
    <div class="container">
        <div class="d-flex justify-content-between mb-3">
            <h3>Manajemen Armada</h3>
            <a href="index.php" class="btn btn-dark">Kembali ke Dashboard</a>
        </div>

        <div class="card mb-4 border-0 shadow-sm">
            <div class="card-header bg-primary text-white fw-bold">Tambah Unit Baru</div>
            <div class="card-body">
                <form method="POST" class="row g-3">
                    <div class="col-md-3"><input type="text" name="nama" class="form-control" placeholder="Nama Unit" required></div>
                    <div class="col-md-2">
                        <select name="jenis" class="form-select">
                            <option value="Bus">Bus</option><option value="Minibus">Minibus</option>
                            <option value="Mobil">Mobil</option><option value="Motor">Motor</option>
                        </select>
                    </div>
                    <div class="col-md-3"><input type="text" name="plat" class="form-control" placeholder="Plat Nomor" required></div>
                    <div class="col-md-2"><input type="number" name="harga" class="form-control" placeholder="Harga Sewa" required></div>
                    <div class="col-md-2"><button name="tambah" class="btn btn-success w-100">Simpan</button></div>
                </form>
            </div>
        </div>

        <div class="card border-0 shadow-sm">
            <div class="card-body p-0">
                <table class="table table-hover mb-0 align-middle">
                    <thead class="table-dark">
                        <tr>
                            <th>Nama Unit</th><th>Jenis</th><th>Plat</th><th>Harga/Hari</th><th>Status</th><th class="text-center">Aksi</th>
                        </tr>
                    </thead>
                    <tbody>
                        <?php
                        $q = mysqli_query($conn, "SELECT * FROM kendaraan");
                        while($k = mysqli_fetch_assoc($q)){
                            $st = ($k['status'] == 'Tersedia') ? 'bg-info' : 'bg-danger';
                        ?>
                        <tr>
                            <td><?= $k['nama'] ?></td>
                            <td><?= $k['jenis'] ?></td>
                            <td><?= $k['plat_nomor'] ?></td>
                            <td>Rp <?= number_format($k['harga_sewa']) ?></td>
                            <td><span class="badge <?= $st ?>"><?= $k['status'] ?></span></td>
                            <td class="text-center">
                                <button class="btn btn-warning btn-sm" data-bs-toggle="modal" data-bs-target="#editK<?= $k['id'] ?>">Edit</button>
                                <a href="hapus.php?id_k=<?= $k['id'] ?>" class="btn btn-danger btn-sm" onclick="return confirm('Yakin hapus?')">Hapus</a>
                            </td>
                        </tr>

                        <div class="modal fade" id="editK<?= $k['id'] ?>" tabindex="-1" aria-hidden="true">
                            <div class="modal-dialog">
                                <form method="POST" class="modal-content">
                                    <div class="modal-header"><h5>Edit <?= $k['nama'] ?></h5></div>
                                    <div class="modal-body">
                                        <input type="hidden" name="id" value="<?= $k['id'] ?>">
                                        <label class="small fw-bold">Nama</label>
                                        <input type="text" name="nama" class="form-control mb-2" value="<?= $k['nama'] ?>">
                                        <label class="small fw-bold">Jenis</label>
                                        <select name="jenis" class="form-select mb-2">
                                            <option value="Bus" <?= $k['jenis']=='Bus'?'selected':'' ?>>Bus</option>
                                            <option value="Minibus" <?= $k['jenis']=='Minibus'?'selected':'' ?>>Minibus</option>
                                            <option value="Mobil" <?= $k['jenis']=='Mobil'?'selected':'' ?>>Mobil</option>
                                            <option value="Motor" <?= $k['jenis']=='Motor'?'selected':'' ?>>Motor</option>
                                        </select>
                                        <label class="small fw-bold">Plat Nomor</label>
                                        <input type="text" name="plat_nomor" class="form-control mb-2" value="<?= $k['plat_nomor'] ?>">
                                        <label class="small fw-bold">Harga Sewa</label>
                                        <input type="number" name="harga_sewa" class="form-control" value="<?= $k['harga_sewa'] ?>">
                                    </div>
                                    <div class="modal-footer">
                                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                                        <button type="submit" name="update" class="btn btn-primary">Simpan</button>
                                    </div>
                                </form>
                            </div>
                        </div>
                        <?php } ?>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
~~~

# 6. Login: #
~~~ php
<?php
include 'config.php';
session_start();

if (isset($_POST['login'])) {
    $user = $_POST['username'];
    $pass = $_POST['password'];

    // Mencari user yang username DAN passwordnya cocok persis
    $query = "SELECT * FROM users WHERE username='$user' AND password='$pass'";
    $res = mysqli_query($conn, $query);

    if (mysqli_num_rows($res) === 1) {
        $_SESSION['admin'] = true;
        header("Location: index.php");
        exit;
    } else {
        $error = true;
    }
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Login Admin</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body { background: #212529; display: flex; align-items: center; height: 100vh; }
        .login-card { max-width: 400px; margin: auto; border-radius: 15px; }
    </style>
</head>
<body>
    <div class="container">
        <div class="card login-card shadow-lg p-4">
            <div class="text-center mb-4">
                <h3 class="fw-bold">ADMIN LOGIN</h3>
                <p class="text-muted small">Sistem Manajemen Sewa</p>
            </div>

            <?php if(isset($error)): ?>
                <div class="alert alert-danger text-center py-2 small">Username atau Password Salah!</div>
            <?php endif; ?>

            <form method="POST">
                <div class="mb-3">
                    <label class="form-label small fw-bold">Username</label>
                    <input type="text" name="username" class="form-control" placeholder="admin" required>
                </div>
                <div class="mb-4">
                    <label class="form-label small fw-bold">Password</label>
                    <input type="password" name="password" class="form-control" placeholder="••••••••" required>
                </div>
                <button name="login" class="btn btn-primary w-100 fw-bold">Masuk Sekarang</button>
            </form>
        </div>
    </div>
</body>
</html>
~~~

# 7. Logout: #
~~~ php
<?php
session_start();
session_destroy();
header("Location: login.php");
?>
~~~

# 8. Nota: #
~~~ php
<?php
include 'config.php';
$id = $_GET['id'];
$q = mysqli_query($conn, "SELECT t.*, p.nama as p_nama, k.nama as k_nama, k.plat_nomor FROM transaksi t 
                         JOIN pelanggan p ON t.id_pelanggan=p.id JOIN kendaraan k ON t.id_kendaraan=k.id WHERE t.id='$id'");
$d = mysqli_fetch_assoc($q);
?>
<!DOCTYPE html>
<html>
<head>
    <title>Nota TRX-<?= $id ?></title>
    <style>
        body { font-family: 'Courier New', Courier, monospace; width: 300px; padding: 20px; }
        .text-center { text-align: center; }
        hr { border-top: 1px dashed black; }
    </style>
</head>
<body onload="window.print()">
    <h3 class="text-center">SEWA APP RENTAL</h3>
    <p class="text-center">Nota Transaksi #TRX-<?= $id ?></p>
    <hr>
    <p>Nama: <?= $d['p_nama'] ?></p>
    <p>Unit: <?= $d['k_nama'] ?> (<?= $d['plat_nomor'] ?>)</p>
    <p>Tgl Sewa: <?= $d['tgl_sewa'] ?></p>
    <p>Tgl Kembali: <?= $d['tgl_kembali'] ?></p>
    <hr>
    <p>Biaya Sewa: Rp <?= number_format($d['total_bayar']) ?></p>
    <p>Denda: Rp <?= number_format($d['denda']) ?></p>
    <p>Metode: <?= $d['metode_bayar'] ?></p>
    <hr>
    <h4 class="text-center">TOTAL: Rp <?= number_format($d['total_bayar'] + $d['denda']) ?></h4>
    <p class="text-center">-- Terima Kasih --</p>
</body>
</html>
~~~

# 9. Pelanggan: #
~~~ php
<?php 
include 'config.php'; 
session_start();
if(!isset($_SESSION['admin'])) { header("Location: login.php"); exit; }

if(isset($_POST['tambah'])){
    $n = $_POST['nama']; $k = $_POST['ktp']; $t = $_POST['telp'];
    mysqli_query($conn, "INSERT INTO pelanggan (nama, ktp, telepon) VALUES ('$n', '$k', '$t')");
    header("Location: pelanggan.php");
}

if(isset($_POST['update'])){
    $id = $_POST['id']; $n = $_POST['nama']; $k = $_POST['ktp']; $t = $_POST['telp'];
    mysqli_query($conn, "UPDATE pelanggan SET nama='$n', ktp='$k', telepon='$t' WHERE id='$id'");
    header("Location: pelanggan.php");
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <title>Master Pelanggan</title>
</head>
<body class="bg-light p-4">
<div class="container">
    <div class="d-flex justify-content-between mb-3"><h3>Data Pelanggan</h3><a href="index.php" class="btn btn-dark">Dashboard</a></div>
    
    <div class="row">
        <div class="col-md-4">
            <div class="card border-0 shadow-sm">
                <div class="card-header bg-dark text-white">Tambah Pelanggan</div>
                <div class="card-body">
                    <form method="POST">
                        <input type="text" name="nama" class="form-control mb-2" placeholder="Nama Lengkap" required>
                        <input type="text" name="ktp" class="form-control mb-2" placeholder="No. KTP" required>
                        <input type="text" name="telp" class="form-control mb-3" placeholder="Telepon" required>
                        <button name="tambah" class="btn btn-primary w-100">Simpan Pelanggan</button>
                    </form>
                </div>
            </div>
        </div>
        <div class="col-md-8">
            <div class="card border-0 shadow-sm p-0">
                <table class="table table-hover mb-0 align-middle">
                    <thead class="table-dark"><tr><th>Nama</th><th>KTP</th><th>Telepon</th><th class="text-center">Aksi</th></tr></thead>
                    <tbody>
                        <?php $q = mysqli_query($conn, "SELECT * FROM pelanggan"); 
                        while($r = mysqli_fetch_assoc($q)){ ?>
                        <tr>
                            <td><?= $r['nama'] ?></td><td><?= $r['ktp'] ?></td><td><?= $r['telepon'] ?></td>
                            <td class="text-center">
                                <button class="btn btn-warning btn-sm" data-bs-toggle="modal" data-bs-target="#editP<?= $r['id'] ?>">Edit</button>
                                <a href="hapus.php?id_p=<?= $r['id'] ?>" class="btn btn-danger btn-sm" onclick="return confirm('Hapus pelanggan?')">Hapus</a>
                            </td>
                        </tr>
                        <div class="modal fade" id="editP<?= $r['id'] ?>" tabindex="-1" aria-hidden="true">
                            <div class="modal-dialog">
                                <form method="POST" class="modal-content">
                                    <div class="modal-header"><h5>Edit Pelanggan</h5></div>
                                    <div class="modal-body">
                                        <input type="hidden" name="id" value="<?= $r['id'] ?>">
                                        <input type="text" name="nama" class="form-control mb-2" value="<?= $r['nama'] ?>">
                                        <input type="text" name="ktp" class="form-control mb-2" value="<?= $r['ktp'] ?>">
                                        <input type="text" name="telp" class="form-control mb-2" value="<?= $r['telepon'] ?>">
                                    </div>
                                    <div class="modal-footer">
                                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                                        <button type="submit" name="update" class="btn btn-primary">Simpan</button>
                                    </div>
                                </form>
                            </div>
                        </div>
                        <?php } ?>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</div>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
~~~

# 10. Transaksi: #
~~~ php
<?php 
session_start();
if(!isset($_SESSION['admin'])) { header("Location: login.php"); exit; }
include 'config.php'; 

// PROSES SEWA
if(isset($_POST['sewa'])){
    $id_k = $_POST['id_kendaraan']; $id_p = $_POST['id_pelanggan'];
    $tgl_s = $_POST['tgl_sewa']; $tgl_k = $_POST['tgl_kembali'];
    $metode = $_POST['metode_bayar'];

    $res_k = mysqli_query($conn, "SELECT harga_sewa FROM kendaraan WHERE id = '$id_k'");
    $dk = mysqli_fetch_assoc($res_k);
    
    $durasi = (strtotime($tgl_k) - strtotime($tgl_s)) / 86400;
    if($durasi <= 0) $durasi = 1;
    $total = $durasi * $dk['harga_sewa'];

    mysqli_query($conn, "INSERT INTO transaksi (id_kendaraan, id_pelanggan, tgl_sewa, tgl_kembali, total_bayar, metode_bayar, status_bayar, status) 
                        VALUES ('$id_k', '$id_p', '$tgl_s', '$tgl_k', '$total', '$metode', 'Lunas', 'Aktif')");
    mysqli_query($conn, "UPDATE kendaraan SET status = 'Disewa' WHERE id = '$id_k'");
    header("Location: transaksi.php");
}

// PENGEMBALIAN & DENDA OTOMATIS
if(isset($_GET['kembali'])){
    $id_t = $_GET['id_t']; $id_k = $_GET['id_k'];
    $tgl_hari_ini = date('Y-m-d');
    
    $res = mysqli_query($conn, "SELECT t.tgl_kembali, k.jenis FROM transaksi t JOIN kendaraan k ON t.id_kendaraan = k.id WHERE t.id = '$id_t'");
    $dt = mysqli_fetch_assoc($res);
    
    $denda = 0;
    if($tgl_hari_ini > $dt['tgl_kembali']){
        $telat = (strtotime($tgl_hari_ini) - strtotime($dt['tgl_kembali'])) / 86400;
        
        // Tarif Denda Sesuai Permintaan
        if($dt['jenis'] == 'Bus') $tarif = 500000;
        elseif($dt['jenis'] == 'Minibus') $tarif = 250000;
        elseif($dt['jenis'] == 'Mobil') $tarif = 150000;
        elseif($dt['jenis'] == 'Motor') $tarif = 100000;
        else $tarif = 50000;
        
        $denda = $telat * $tarif;
    }

    mysqli_query($conn, "UPDATE transaksi SET status = 'Selesai', denda = '$denda' WHERE id = '$id_t'");
    mysqli_query($conn, "UPDATE kendaraan SET status = 'Tersedia' WHERE id = '$id_k'");
    header("Location: transaksi.php");
}
?>
<!DOCTYPE html>
<html lang="id"><head><link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet"></head>
<body class="bg-light p-4"><div class="container">
    <div class="d-flex justify-content-between mb-4"><h3>Penyewaan & Pengembalian</h3><a href="index.php" class="btn btn-dark">Dashboard</a></div>
    <div class="row">
        <div class="col-md-4">
            <div class="card shadow-sm border-0">
                <div class="card-header bg-primary text-white">Input Sewa</div>
                <div class="card-body">
                    <form method="POST">
                        <label class="small fw-bold">Unit</label>
                        <select name="id_kendaraan" class="form-select mb-2">
                            <?php $vk=mysqli_query($conn,"SELECT * FROM kendaraan WHERE status='Tersedia'"); 
                            while($rk=mysqli_fetch_assoc($vk)) echo "<option value='{$rk['id']}'>{$rk['nama']} ({$rk['jenis']})</option>"; ?>
                        </select>
                        <label class="small fw-bold">Pelanggan</label>
                        <select name="id_pelanggan" class="form-select mb-2">
                            <?php $vp=mysqli_query($conn,"SELECT * FROM pelanggan"); 
                            while($rp=mysqli_fetch_assoc($vp)) echo "<option value='{$rp['id']}'>{$rp['nama']}</option>"; ?>
                        </select>
                        <input type="date" name="tgl_sewa" class="form-control mb-2" value="<?=date('Y-m-d')?>">
                        <input type="date" name="tgl_kembali" class="form-control mb-2" required>
                        <select name="metode_bayar" class="form-select mb-3"><option value="Tunai">Tunai</option><option value="Transfer">Transfer</option></select>
                        <button name="sewa" class="btn btn-success w-100">Proses</button>
                    </form>
                </div>
            </div>
        </div>
        <div class="col-md-8">
            <div class="card shadow-sm border-0">
                <div class="card-header bg-warning fw-bold">Stok Disewa (Aktif)</div>
                <div class="card-body p-0">
                    <table class="table table-hover mb-0">
                        <thead><tr><th>Unit</th><th>Penyewa</th><th>Batas</th><th>Aksi</th></tr></thead>
                        <tbody>
                            <?php $qt=mysqli_query($conn,"SELECT t.*, k.nama as k_nama, p.nama as p_nama FROM transaksi t JOIN kendaraan k ON t.id_kendaraan=k.id JOIN pelanggan p ON t.id_pelanggan=p.id WHERE t.status='Aktif'");
                            while($rt=mysqli_fetch_assoc($qt)) echo "<tr><td>{$rt['k_nama']}</td><td>{$rt['p_nama']}</td><td>{$rt['tgl_kembali']}</td>
                            <td><a href='transaksi.php?kembali=1&id_t={$rt['id']}&id_k={$rt['id_kendaraan']}' class='btn btn-danger btn-sm' onclick='return confirm(\"Selesaikan sewa?\")'>Terima Kembali</a></td></tr>"; ?>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>
</div></body></html>
~~~







