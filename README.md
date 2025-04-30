import os
from pdf2image import convert_from_path
import pytesseract
from collections import Counter
import pandas as pd

# CAMINHO DO TESSERACT
pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files\Tesseract-OCR\tesseract.exe'

# CAMINHO DOS PDFs
pasta_pdfs = r'C:\Users\Lucas\Desktop\PROGRAMAS\analise de dados MS\OCR\pdfss'
# CAMINHO DO EXCEL DE SAÍDA
saida_excel = r'C:\Users\Lucas\Desktop\PROGRAMAS\analise de dados MS\OCR\resultado_ocr.xlsx'

def extrair_solo_ocr(caminho_pdf):
    imagens = convert_from_path(caminho_pdf, dpi=300)
    texto = ""
    for img in imagens:
        texto += pytesseract.image_to_string(img, lang='por') + "\n"

    linhas_camadas = []
    for linha in texto.splitlines():
        if any(palavra in linha.lower() for palavra in ["argila", "areia", "silte", "cascalho"]):
            linhas_camadas.append(linha.strip().lower())

    if not linhas_camadas:
        return "Não foi possível identificar camadas."

    contagem = Counter()
    for linha in linhas_camadas:
        if "argila arenosa" in linha:
            contagem["Argila arenosa"] += 1
        elif "argila siltosa" in linha:
            contagem["Argila siltosa"] += 1
        elif "silte" in linha:
            contagem["Silte"] += 1
        elif "areia" in linha:
            contagem["Areia"] += 1
        elif "argila" in linha:
            contagem["Argila"] += 1
        elif "cascalho" in linha:
            contagem["Cascalho"] += 1

    if not contagem:
        return "Nenhum tipo de solo identificado nas camadas."

    return contagem.most_common(1)[0][0]

relatorios = []
for arquivo in os.listdir(pasta_pdfs):
    if arquivo.lower().endswith('.pdf'):
        caminho_pdf = os.path.join(pasta_pdfs, arquivo)
        solo = extrair_solo_ocr(caminho_pdf)
        relatorios.append((arquivo, solo))

df = pd.DataFrame(relatorios, columns=["Relatório", "Solo Predominante (OCR)"])
df.to_excel(saida_excel, index=False)
print("Análise concluída! Resultado salvo em:", saida_excel)
