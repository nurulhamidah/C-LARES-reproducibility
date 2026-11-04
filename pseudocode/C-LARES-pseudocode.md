FUNGSI C_LARES(user_target, semua_user, rating_matrix, lokasi_matrix, alpha, k_range)

    // ============================================
    // TAHAP 0: Cari k optimal menggunakan Silhouette Coefficient
    // ============================================
    skor_terbaik = -1
    k_optimal = NULL
    
    UNTUK setiap k DALAM k_range:    // mis. k = [2, 3, 4, ..., 10]
        cluster_hasil = K_MEANS(semua_user, fitur_gabungan(rating_matrix, lokasi_matrix), k)
        
        total_S = 0
        UNTUK setiap user_i DALAM semua_user:
            S_i = SILHOUETTE_COEFFICIENT(user_i, cluster_hasil, cluster_hasil[user_i])
            total_S += S_i
        
        rata2_S = total_S / jumlah(semua_user)   // rata-rata Silhouette seluruh dataset
        
        JIKA rata2_S > skor_terbaik:
            skor_terbaik = rata2_S
            k_optimal = k
    

