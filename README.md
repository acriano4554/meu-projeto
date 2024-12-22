# Exemplo de uma aplicação simples de loja online que aceita pagamento via Pix
# Este exemplo usa Flask para o backend

from flask import Flask, request, jsonify
import qrcode
from io import BytesIO
import base64

app = Flask(__name__)
.+

# Dados fictícios da loja
LOJA_NOME = "Celly_Moda.Online"
CHAVE_PIX = "694.008.492-49"

# Produtos disponíveis
PRODUTOS = {
    1: {"nome": "Produto A", "preco": 50.00},
    2: {"nome": "Produto B", "preco": 30.00},
    3: {"nome": "Produto C", "preco": 20.00},
}

@app.route("/produtos", methods=["GET"])
def listar_produtos():
    return jsonify(PRODUTOS)

@app.route("/comprar/<int:produto_id>", methods=["POST"])
def comprar(produto_id):
    produto = PRODUTOS.get(produto_id)
    if not produto:
        return jsonify({"erro": "Produto não encontrado."}), 404

    valor = produto["preco"]
                
    # Gerar o código Pix (fictício neste exemplo)
    codigo_pix = f"00020126330014br.gov.bcb.pix0114{CHAVE_PIX}520400005303986540{int(valor * 100):02d}5802BR5913{LOJA_NOME}6009SAO PAULO62070503***6304"

    # Gerar o QR Code a partir do código Pix
    qr = qrcode.QRCode(
        version=1,
        error_correction=qrcode.constants.ERROR_CORRECT_L,
        box_size=10,
        border=4,
    )
    qr.add_data(codigo_pix)
    qr.make(fit=True)

    img = qr.make_image(fill_color="black", back_color="white")
    buffered = BytesIO()
    img.save(buffered, format="PNG")
    qr_code_base64 = base64.b64encode(buffered.getvalue()).decode("utf-8")

    return jsonify({
        "produto": produto,
        "valor": valor,
        "codigo_pix": codigo_pix,
        "qr_code": qr_code_base64
    })

if __name__ == "__main__":
    app.run(debug=True)
