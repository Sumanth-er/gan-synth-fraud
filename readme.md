from llama_index.core import SimpleDirectoryReader, VectorStoreIndex
from llama_index.core.node_parser import SentenceSplitter
from llama_index.embeddings.huggingface import HuggingFaceEmbedding
from llama_index.vector_stores.faiss import FaissVectorStore

# 1. Load PDFs using SimpleDirectoryReader
documents = SimpleDirectoryReader("./pdfs").load_data()  # <-- your folder path

# 2. Split into chunks (nodes)
parser = SentenceSplitter(chunk_size=512, chunk_overlap=50)
nodes = parser.get_nodes_from_documents(documents)

# Print chunks
print("🔹 Chunks:")
for i, node in enumerate(nodes[:5]):  # just show first 5 chunks
    print(f"--- Chunk {i+1} ---")
    print(node.text[:300], "...\n")  # print first 300 chars of each chunk
    print("Metadata:", node.metadata)
    print()

# 3. Create embedding model
embed_model = HuggingFaceEmbedding(model_name="sentence-transformers/all-MiniLM-L6-v2")

# 4. Print embeddings for first few nodes
print("🔹 Embeddings:")
for i, node in enumerate(nodes[:3]):
    emb = embed_model.get_text_embedding(node.text)
    print(f"Embedding {i+1} (dim={len(emb)}):", emb[:10], "...")  # show first 10 dims
    print()

# 5. Create vector index
vector_store = FaissVectorStore()
index = VectorStoreIndex(nodes, embed_model=embed_model, vector_store=vector_store)

# 6. Print vector index info
print("🔹 Vector Index Info:")
print("Number of vectors stored:", vector_store.faiss_index.ntotal)
