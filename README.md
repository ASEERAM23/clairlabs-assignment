import openai
from pydantic import BaseModel, ValidationError
from typing import Dict, Any

class G2PEvidence(BaseModel):
    gene_symbol: str = Field(..., description="Normalized HGNC Gene Symbol")
    variant: str = Field(..., description="Variant notation, e.g., V600E")
    phenotype: str = Field(..., description="Disease or condition")
    evidence_level: str
    source: str

def ai_normalize_gene(raw_gene: str) -> str:
    """
    Simulates using an LLM to normalize non-standard gene names.
    e.g., 'HER2' -> 'ERBB2'
    """
    # In production, you'd use an OpenAI/Bedrock call here.
    mapping = {"HER2": "ERBB2", "p53": "TP53"}
    return mapping.get(raw_gene.upper(), raw_gene.upper())

def process_raw_record(raw_data: Dict[str, Any]) -> G2PEvidence:
    try:
        # 1. Step: Basic extraction
        gene = raw_data.get("gene", "Unknown")
        
        # 2. Step: AI/Lookup Normalization
        normalized_gene = ai_normalize_gene(gene)
        
        # 3. Step: Validation via Pydantic
        record = G2PEvidence(
            gene_symbol=normalized_gene,
            variant=raw_data.get("variant_description", "N/A"),
            phenotype=raw_data.get("condition", "Unknown"),
            evidence_level=raw_data.get("level", "Tier III"),
            source=raw_data.get("source_id", "Internal")
        )
        return record
    except ValidationError as e:
        print(f"Data Quality Error: {e}")
        raise

# Example Usage
raw_input = {"gene": "HER2", "variant_description": "V600E", "condition": "Breast Cancer"}
final_record = process_raw_record(raw_input)
print(final_record.json())
