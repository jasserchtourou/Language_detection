# Language Detection Model
![image](https://github.com/jasserchtourou/Language_detection/assets/124272855/399d807a-0b58-4a21-baf7-9ad97b5271f1)


This repository contains a PyTorch-based model for language identification using multiple language detection methods. It combines predictions from various language detection libraries and models to determine the most probable language for a given text input.

## Overview
The LanguageIdentificationModel integrates scores from several language detection methods:

LangDetect: Language detection using the langdetect library.
LangID: Language identification with the langid library.
Hugging Face: Language classification using the papluca/xlm-roberta-base-language-detection model from Hugging Face Transformers.
FastText: Language prediction using the lid.176.bin model from FastText.
These methods are integrated into a PyTorch neural network model, enabling accurate language identification across various text inputs.

---
language:
- en
- ar
- fr
- es
- pt
- ja
- it
- de
- ru
- zh
metrics:
- accuracy
- code_eval
library_name: transformers
pipeline_tag: text-classification

## Installation
Clone this repository:
      git clone https://github.com/jasserchtourou/Language_detection.git
      cd Language_detection
      
Install the required dependencies:
      pip install -r requirements.txt

Download the necessary models:
      Download the papluca/xlm-roberta-base-language-detection model from Hugging Face Transformers.
      Download the lid.176.bin model from FastText and place it in the repository.

## Usage
Training the Model
    To train the model, customize the input_size, hidden_size, and output_size in train.py based on your data and run:
          python train.py

## Using the Model
After training, you can use the model for language identification:
    from language_identification_model import LanguageIdentificationModel

    model = LanguageIdentificationModel(input_size, hidden_size, output_size)
    model.load_state_dict(torch.load('model_weights.bin'))
    model.eval()
    text = "Bonjour tout le monde"
    language = model.predict(text)
    print(f"The identified language is: {language}")



## Contributing
Contributions are welcome! If you have any suggestions, improvements, or bug fixes, please submit a pull request or open an issue.

## License
This project is licensed under the MIT License - see the LICENSE file for details.

![image](https://github.com/jasserchtourou/Language_detection/assets/124272855/35c1d9b0-5a8a-43d8-b5c7-f5a082722812)

