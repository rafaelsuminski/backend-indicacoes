import express from "express";
import cors from "cors";
import admin from "firebase-admin";
import { readFileSync } from "fs";

// Carregar credenciais do Firebase
const serviceAccount = JSON.parse(readFileSync("caminho/para/sua-chave.json", "utf-8"));

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
});

const db = admin.firestore();
const app = express();
app.use(cors());
app.use(express.json());

// Criar nova indicação
app.post("/indicacoes", async (req, res) => {
  try {
    const { nome, telefone, endereco, produtos, observacao, status } = req.body;
    const docRef = await db.collection("indicacoes").add({ nome, telefone, endereco, produtos, observacao, status: status || "Indicação" });
    res.status(201).json({ id: docRef.id });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Obter todas as indicações
app.get("/indicacoes", async (req, res) => {
  try {
    const snapshot = await db.collection("indicacoes").get();
    const indicacoes = snapshot.docs.map((doc) => ({ id: doc.id, ...doc.data() }));
    res.json(indicacoes);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Atualizar status da indicação
app.put("/indicacoes/:id", async (req, res) => {
  try {
    const { id } = req.params;
    const { status } = req.body;
    await db.collection("indicacoes").doc(id).update({ status });
    res.json({ message: "Status atualizado com sucesso!" });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Iniciar servidor
const PORT = 5000;
app.listen(PORT, () => console.log(`Servidor rodando na porta ${PORT}`));
