📁 File: app.py

from fastapi import FastAPI, File, UploadFile, Form from fastapi.middleware.cors import CORSMiddleware from pdf_utils import process_pdf, ask_question from tempfile import NamedTemporaryFile import shutil

app = FastAPI()

app.add_middleware( CORSMiddleware, allow_origins=[""], allow_credentials=True, allow_methods=[""], allow_headers=["*"], )

pdf_store = {}

@app.post("/upload_pdf/") async def upload_pdf(file: UploadFile = File(...)): with NamedTemporaryFile(delete=False) as tmp: shutil.copyfileobj(file.file, tmp) pdf_id = file.filename chunks, vectorstore = process_pdf(tmp.name) pdf_store[pdf_id] = vectorstore return {"message": "PDF uploaded successfully", "pdf_id": pdf_id}

@app.post("/ask_question/") async def ask(pdf_id: str = Form(...), question: str = Form(...)): if pdf_id not in pdf_store: return {"error": "PDF not found"} vectorstore = pdf_store[pdf_id] answer = ask_question(vectorstore, 
question) return {"answer": answer}
