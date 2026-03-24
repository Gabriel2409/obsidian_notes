#linux 
 - Press `Win + R` to open the Run dialog, type `certmgr.msc`, and press Enter. This will open the Certificate Manager.

- Go to `Trusted Root Certification > Certificates` and double click on `Zscaler Root CA`

- Export it in the `DER encoded binary X.509 (.CER)` format and place it in WSL (for ex in your home folder).

- Convert it to pem: `openssl x509 -inform DER -in <path_to_certificate.cer> -out zscaler-root-ca.pem` (replace `<path_to_certificate.cer>`)

- Move it to `/etc/ssl/certs/zscaler-root-ca.pem`

- Run `sudo update-ca-certificates`

Note: for some, instead,

`openssl x509 -inform DER -in <path_to_certificate.cer> -out zscaler-root-ca.crt`

and move both crt and cer in `/usr/local/share/ca-certificates/` before running `sudo update-ca-certificates`
