"""
Game Ular (Snake) Sederhana
---------------------------
Contoh game sederhana menggunakan pygame (pygame-ce) yang mendemonstrasikan:
- Loop utama game (game loop)
- Penanganan input keyboard
- Deteksi tabrakan (collision detection)
- Skor & level kecepatan
- Layar "Game Over" dengan opsi main lagi

Cara menjalankan:
    pip install pygame-ce
    python game_ular.py

Kontrol:
    Panah Atas/Bawah/Kiri/Kanan  -> menggerakkan ular
    R (saat Game Over)          -> main lagi
    Esc / tombol close jendela  -> keluar
"""

import random
import sys

import pygame

# ----------------------------- Konfigurasi ----------------------------- #
LEBAR_LAYAR = 600
TINGGI_LAYAR = 400
UKURAN_SEL = 20

WARNA_LATAR = (15, 15, 20)
WARNA_ULAR_KEPALA = (0, 200, 0)
WARNA_ULAR_BADAN = (0, 150, 0)
WARNA_MAKANAN = (220, 60, 60)
WARNA_TEKS = (240, 240, 240)
WARNA_GARIS = (30, 30, 35)

FPS_AWAL = 8
FPS_MAKS = 20


class Ular:
    """Merepresentasikan ular yang dikendalikan pemain."""

    def __init__(self):
        tengah_x = (LEBAR_LAYAR // UKURAN_SEL // 2) * UKURAN_SEL
        tengah_y = (TINGGI_LAYAR // UKURAN_SEL // 2) * UKURAN_SEL
        self.segmen = [(tengah_x, tengah_y)]
        self.arah = (UKURAN_SEL, 0)  # bergerak ke kanan di awal
        self.arah_berikutnya = self.arah
        self.tumbuh = False

    def ubah_arah(self, arah_baru):
        # Mencegah ular berbalik 180 derajat langsung ke badannya sendiri
        berlawanan = (-arah_baru[0], -arah_baru[1])
        if berlawanan != self.arah:
            self.arah_berikutnya = arah_baru

    def gerak(self):
        self.arah = self.arah_berikutnya
        kepala_x, kepala_y = self.segmen[0]
        dx, dy = self.arah
        kepala_baru = (kepala_x + dx, kepala_y + dy)
        self.segmen.insert(0, kepala_baru)
        if not self.tumbuh:
            self.segmen.pop()
        else:
            self.tumbuh = False

    def makan(self):
        self.tumbuh = True

    def cek_tabrakan_diri(self):
        kepala = self.segmen[0]
        return kepala in self.segmen[1:]

    def cek_keluar_layar(self):
        kepala_x, kepala_y = self.segmen[0]
        return (
            kepala_x < 0
            or kepala_x >= LEBAR_LAYAR
            or kepala_y < 0
            or kepala_y >= TINGGI_LAYAR
        )

    def gambar(self, layar):
        for i, (x, y) in enumerate(self.segmen):
            warna = WARNA_ULAR_KEPALA if i == 0 else WARNA_ULAR_BADAN
            pygame.draw.rect(layar, warna, (x, y, UKURAN_SEL, UKURAN_SEL))
            pygame.draw.rect(layar, WARNA_LATAR, (x, y, UKURAN_SEL, UKURAN_SEL), 1)

def posisi_makanan_baru(ular):
    """Membuat posisi makanan acak yang tidak menimpa badan ular."""
    kolom_maks = LEBAR_LAYAR // UKURAN_SEL
    baris_maks = TINGGI_LAYAR // UKURAN_SEL
    while True:
        x = random.randint(0, kolom_maks - 1) * UKURAN_SEL
        y = random.randint(0, baris_maks - 1) * UKURAN_SEL
        if (x, y) not in ular.segmen:
            return (x, y)

def gambar_grid(layar):
    for x in range(0, LEBAR_LAYAR, UKURAN_SEL):
        pygame.draw.line(layar, WARNA_GARIS, (x, 0), (x, TINGGI_LAYAR))
    for y in range(0, TINGGI_LAYAR, UKURAN_SEL):
        pygame.draw.line(layar, WARNA_GARIS, (0, y), (LEBAR_LAYAR, y))

def tampilkan_teks(layar, font, teks, warna, pos, tengah=False):
    permukaan = font.render(teks, True, warna)
    rect = permukaan.get_rect()
    if tengah:
        rect.center = pos
    else:
        rect.topleft = pos
    layar.blit(permukaan, rect)

def main():
    pygame.init()
    layar = pygame.display.set_mode((LEBAR_LAYAR, TINGGI_LAYAR))
    pygame.display.set_caption("Game Ular Sederhana")
    jam = pygame.time.Clock()
    font_skor = pygame.font.SysFont("consolas", 22)
    font_besar = pygame.font.SysFont("consolas", 40, bold=True)
    font_kecil = pygame.font.SysFont("consolas", 20)

    while True:  # loop untuk memungkinkan main lagi setelah game over
        ular = Ular()
        makanan = posisi_makanan_baru(ular)
        skor = 0
        fps = FPS_AWAL
        game_over = False

        while True:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_ESCAPE:
                        pygame.quit()
                        sys.exit()
                    if not game_over:
                        if event.key in (pygame.K_UP, pygame.K_w):
                            ular.ubah_arah((0, -UKURAN_SEL))
                        elif event.key in (pygame.K_DOWN, pygame.K_s):
                            ular.ubah_arah((0, UKURAN_SEL))
                        elif event.key in (pygame.K_LEFT, pygame.K_a):
                            ular.ubah_arah((-UKURAN_SEL, 0))
                        elif event.key in (pygame.K_RIGHT, pygame.K_d):
                            ular.ubah_arah((UKURAN_SEL, 0))
                    else:
                        if event.key == pygame.K_r:
                            game_over = "restart"

            if game_over == "restart":
                break  # keluar ke loop luar untuk memulai game baru

            if not game_over:
                ular.gerak()

                if ular.cek_keluar_layar() or ular.cek_tabrakan_diri():
                    game_over = True

                if ular.segmen[0] == makanan:
                    ular.makan()
                    skor += 1
                    makanan = posisi_makanan_baru(ular)
                    fps = min(FPS_MAKS, FPS_AWAL + skor // 3)

            # ------------------------- Menggambar ------------------------- #
            layar.fill(WARNA_LATAR)
            gambar_grid(layar)
            pygame.draw.rect(
                layar, WARNA_MAKANAN, (*makanan, UKURAN_SEL, UKURAN_SEL)
            )
            ular.gambar(layar)
            tampilkan_teks(
                layar, font_skor, f"Skor: {skor}", WARNA_TEKS, (10, 10)
            )

            if game_over is True:
                tampilkan_teks(
                    layar,
                    font_besar,
                    "GAME OVER",
                    (255, 80, 80),
                    (LEBAR_LAYAR // 2, TINGGI_LAYAR // 2 - 20),
                    tengah=True,
                )
                tampilkan_teks(
                    layar,
                    font_kecil,
                    "Tekan R untuk main lagi atau Esc untuk keluar",
                    WARNA_TEKS,
                    (LEBAR_LAYAR // 2, TINGGI_LAYAR // 2 + 20),
                    tengah=True,
                )

            pygame.display.flip()
            jam.tick(fps)

if __name__ == "__main__":
    main()
