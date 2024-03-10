#SRC #Sem1 #BSR #wyklad

HTTP/1.0 Basic Authentication (RFC 2616)
- Zasób może być chroniony, aby uzyskać do niego dostęp trzeba się uwieżytelnić
- Przy próbie dostępu do zasoby wyświetla okno z logowaniem
- Jedyne dane jakimi można się uwieżytelnić to username i hasło
- Hasło przesyłane w formacie Base64 przez nagłówek **Authorization**
- Hasło nie jest zabezpieczonego _confidentiality_, jako że jest ono zakodowane ale nie szyfrowane
- Przestażały

HTTP/1.1 Digest Authentication (RFC 2617)
- Klient Web _challange_-uje serwer przez nocne (?)
- Proces składa się z potrójnego _hashowania_:
	- _Hash1_ => MD5 (username:realm:password)
	- _Hash2_ => MD5 (HTTP_method:URL)
	- _Hash3_ => MD5 (_Hash1_:nocne:_Hash2_)
- Przesyłany jest _Hash3_ więc hasło nie jest łatwo dostępne
- Przestażały
- Wariancja która zapobiega _nonce replay attacks_ to przechowywanie po stronie serwera nonce z _timestamp_-ami i ignorowanie _nonce_ starszych jak jakiś _treshold_
- #TODO Na co podatny jest HTTP Digesta Authentication (nie crypto attaks)
- Otwarty na inne metody uwieżytelniania klienta (np. Amazon S3 lub OAuth )

HTTP/2.0
- SPDY transport protocol with optional [[TLS]] profile

HTTP/3.0
- [[QUIC]] protocol with obligatory TLS 1.3