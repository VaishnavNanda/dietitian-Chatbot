


## AIM:
The aim of a dietitian chatbot is to provide personalized nutrition advice, promote healthy eating habits, and support dietary goals.

## PROCEDURE:
# Step 1: Read CSV File with Error Handling for Encoding Issues

The function extract_text_from_csv reads a CSV file, handling potential encoding issues by first trying 'utf-8' and falling back to 'latin1' if needed. It then converts each row of the CSV file into a sentence describing the food item and its nutrition facts, and combines all rows into a single text block.

# Step 2: Clean the Extracted Text

The clean_text function removes unnecessary spaces, replacing multiple spaces with a single one, and trims any leading or trailing whitespace.

# Step 3: Split Text into Sentences

The split_text function splits the cleaned text into individual sentences using NLTK’s sent_tokenize.

# Step 4: Create Sentence Embeddings

The create_embeddings function takes these sentences and creates embeddings using the SentenceTransformer model, which encodes them into vector representations.

# Step 5: Build FAISS Index for Efficient Search

The build_faiss_index function normalizes the embeddings and adds them to a FAISS index, enabling efficient similarity searches using cosine similarity.

# Step 6: Retrieve Similar Sentences Based on Query

The retrieve_similar_sentences function creates an embedding for the user's query and uses FAISS to search for the top 3 most similar sentences from the index, applying a similarity threshold to filter relevant results.
# Step 7: Curate the Response

The curate_response function generates responses in different formats (summarization, detailed, or insights) based on the user’s preference, organizing and presenting relevant information.

# Step 8: Chatbot Interaction

The main function initializes the chatbot, processes user queries, and calls the above functions to provide relevant responses, looping until the user types 'exit'.

# PROGRAM :
```  python
import pandas as pd
import re
from sentence_transformers import SentenceTransformer
import faiss
# Import NLTK
import nltk

# Download the 'punkt' resource for sentence tokenization
nltk.download('punkt')

from nltk.tokenize import sent_tokenize
from transformers import pipeline
# Step 1: Read CSV File with error handling for encoding issues
def extract_text_from_csv(csv_file):
    try:
        df = pd.read_csv(csv_file, encoding='utf-8')
    except UnicodeDecodeError:
        df = pd.read_csv(csv_file, encoding='latin1')

    # Convert each row into a sentence with food item and its nutrition facts
    df['text'] = df.apply(lambda row: f"{row['food']} contains {row['calories']} calories, {row['proteins']} protein, "
                                      f"{row['fat']} fat, {row['carbhohydrates']} carbohydrates, {row['fiber']} fiber, "
                                      f"vitamins: {row['vitamins']}, and minerals: {row['minerals']}.", axis=1)
    text = ' '.join(df['text'].tolist())  # Combine all rows into one text block
    return text

def clean_text(text):
    text = re.sub(r'\s+', ' ', text)  # Replace multiple spaces with a single space
    return text.strip()

def split_text(text):
    return sent_tokenize(text)  # Split text into sentences using NLTK

def create_embeddings(text_chunks, model):
    embeddings = model.encode(text_chunks, convert_to_numpy=True)  # Create embeddings for each text chunk
    return embeddings

def build_faiss_index(embeddings):
    dimension = embeddings.shape[1]  # Get the dimensionality of the embeddings
    index = faiss.IndexFlatIP(dimension)  # Using Inner Product (Cosine Similarity)
    faiss.normalize_L2(embeddings)  # Normalize embeddings for cosine similarity
    index.add(embeddings)  # Add the embeddings to the index
    return index

def retrieve_similar_sentences(query, model, index, text_chunks, threshold=0.5):
    query_embedding = model.encode([query], convert_to_numpy=True)  # Create an embedding for the query
    faiss.normalize_L2(query_embedding)  # Normalize the query embedding for cosine similarity
    distances, indices = index.search(query_embedding, 3)  # Search for the top 3 nearest embeddings
    results = [text_chunks[i] for i, distance in zip(indices[0], distances[0]) if distance > threshold]  # Apply threshold
    return results if results else ["No relevant sentences found."]

def curate_response(similar_sentences, system_message):
    if system_message == "summarize":
        summarizer = pipeline("summarization")  # Use a summarization model
        summarized_text = summarizer(' '.join(similar_sentences), max_length=130, min_length=30, do_sample=False)
        return summarized_text[0]['summary_text']

    elif system_message == "detailed response":
        return '\n\n'.join(similar_sentences)

    elif system_message == "insights only":
        insights = [sentence for sentence in similar_sentences if sentence.startswith('-')]  # Extract bullet points
        return '\n'.join(insights) if insights else "No key insights found."

    else:
        return '\n\n'.join(similar_sentences)

def main():
    # Step 1: Read and Extract text from a CSV file
    csv_file = '/content/888.csv'  # Modify to your actual CSV file path
    raw_text = extract_text_from_csv(csv_file)

    # Step 2: Clean and preprocess the text
    cleaned_text = clean_text(raw_text)

    # Step 3: Split the text into sentences
    text_chunks = split_text(cleaned_text)

    # Step 4: Load the embedding model and create embeddings for each sentence
    model = SentenceTransformer('paraphrase-mpnet-base-v2')  # A larger, more robust model for better embeddings
    embeddings = create_embeddings(text_chunks, model)

    # Step 5: Build the FAISS index
    index = build_faiss_index(embeddings)

    print("Welcome to the Dietitian Chatbot!")
    print("Ask about the nutrition content of food items, or type 'exit' to quit.")
    system_message = 'detailed response'

    while True:
        query = input("\nYou: ")
        if query.lower() == 'exit':
            break

        # Step 6: Retrieve relevant sentences based on the query
        similar_sentences = retrieve_similar_sentences(query, model, index, text_chunks)

        # Step 7: Curate the response based on system message
        curated_response = curate_response(similar_sentences, system_message)

        # Display result
        print(f"Bot:\n{curated_response}\n")

if __name__ == "__main__":
    main()
```
## OUTPUT:

![Screenshot (38)](https://github.com/user-attachments/assets/39b31d4d-a994-4bf8-ab57-a63086253b90)

## RESULT:

A chatbot that answers nutrition queries by retrieving relevant information from a CSV file and using FAISS for search.




