---
sidebar_position: 2
---

# How can we ask AlphaFold 3 to predict the structure of biomolecules?

The simplest way to input sequences for AlphaFold 3 to predict is to use the webapp to manually enter protein and nucleic acid sequences, post-translational modifications, ligands, etc.

However, there are some downsides to this:
- You will have no local record of your input to AlphaFold 3, which hinders reproducibility
- It is time-consuming to manually enter information for many complexes
- Mistakes and discrepencies may occur when manually entering information

Instead, it is good practice to programatically generate JSON files which can then be uploaded to the AlphaFold 3 webserver.
If you are not familiar with the JSON file format, then please check out [this brief w3schools introduction to JSON files.](https://www.w3schools.com/js/js_json_intro.asp)

### Getting Set Up

So, what will we predict?
To show off the capabilities of AlphaFold 3, here are an assortment of structures that may be predicted:

*monomeric protein*
Components: GTPase HRas.
A human enzyme which regulates cell division in response to stimulation by growth factors.
GTPase HRas sequence: MTEYKLVVVGAGGVGKSALTIQLIQNHFVDEYDPTIEDSYRKQVVIDGETCLLDILDTAGQEEYSAMRDQYMRTGEGFLCVFAINNTKSFEDIHQYREQIKRVKDSDDVPMVLVGNKCDLAARTVESRQAQDLARSYGIPYIETSAKTRQGVEDAFYTLVREIRQHKLRKLNPPDESGPGCMSCKCVLS

*protein complex*
Components: p53 and MDM2.
p53 is a tumour suppression protein which prevents cancer formation.
MDM2 is a protein which binds and directly inhibits p53. 

p53 sequence: MEEPQSDPSVEPPLSQETFSDLWKLLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPRMPEAAPPVAPAPAAPTPAAPAPAPSWPLSSSVPSQKTYQGSYGFRLGFLHSGTAKSVTCTYSPALNKMFCQLAKTCPVQLWVDSTPPPGTRVRAMAIYKQSQHMTEVVRRCPHHERCSDSDGLAPPQHLIRVEGNLRVEYLDDRNTFRHSVVVPYEPPEVGSDCTTIHYNYMCNSSCMGGMNRRPILTIITLEDSSGNLLGRNSFEVRVCACPGRDRRTEEENLRKKGEPHHELPPGSTKRALPNNTSSSPQPKKKPLDGEYFTLQIRGRERFEMFRELNEALELKDAQAGKEPGGSRAHSSHLKSKKGQSTSRHKKLMFKTEGPDSD
MDM2 sequence: MCNTNMSVPTDGAVTTSQIPASEQETLVRPKPLLLKLLKSVGAQKDTYTMKEVLFYLGQYIMTKRLYDEKQQHIVYCSNDLLGDLFGVPSFSVKEHRKIYTMIYRNLVVVNQQESSDSGTSVSENRCHLEGGSDQKDLVQELQEEKPSSSHLVSRPSTSSRRRAISETEENSDELSGERQRKRHKSDSISLSFDESLALCVIREICCERSSSSESTGTPSNPDLDAGVSEHSGDWLDQDSVSDQFSVEFEVESLDSEDYSLSEEGQELSDEDDEVYQVTVYQAGESDTDSFEEDPEISLADYWKCTSCNEMNPPLPSHCNRCWALRENWLPEDKGKDKGEISEKAKLENSTQAEEGFDVPDCKKTIVNDSRESCVEENDDKITQASQSQESEDYSQPSTSSSIIYSSQEDVKEFEREETQDKEESVESSLPLNAIEPCVICQGRPKNGCIVHGKTGHLMACFTCAKKLKKRNKPCPVCRQPIQMIVLTYFP

*multimeric protein-ligand complex*
Components: hemoglobin subunit alpha x2, hemoglobin subunit beta x2, heme x4
Hemoglobin subunit alpha is one of the two types of polypeptide chains in hemoglobin, responsible for binding oxygen via its heme group and forming part of the tetrameric hemoglobin structure.
Hemoglobin subunit beta is the second type of polypeptide chain in hemoglobin, also responsible for binding oxygen via its heme group.
Heme is an iron-containing porphyrin ligand that binds oxygen in proteins like hemoglobin, facilitating oxygen transport and redox reactions.

Hemoglobin alpha sequence: MVLSPADKTNVKAAWGKVGAHAGEYGAEALERMFLSFPTTKTYFPHFDLSHGSAQVKGHGKKVADALTNAVAHVDDMPNALSALSDLHAHKLRVDPVNFKLLSHCLLVTLAAHLPAEFTPAVHASLDKFLASVSTVLTSKYR
Hemoglobin beta sequence: MVHLTPEEKSAVTALWGKVNVDEVGGEALGRLLVVYPWTQRFFESFGDLSTPDAVMGNPKVKAHGKKVLGAFSDGLAHLDNLKGTFATLSELHCDKLHVDPENFRLLGNVLVCVLAHHFGKEFTPPVQAAYQKVVAGVANALAHKYH
Heme does not have a sequence but is one of the ligands supported by AlphaFold 3.


### What is Structural Prediction?

Protein structure prediction determines the 3D structure of a protein from its amino acid sequence. The function of a protein
is closely linked to its shape, or conformation, so understanding the structure provides insights into how proteins perform various functions
such as catalysing reactions or interacting with binding partners. Understanding protein structure is also an important aspect of
drug discovery and development.

### Why Predict Protein Structure?

Protein structure prediction has several applications:

- **Drug Design**: Understanding protein conformation helps in designing drugs that interact target proteins.
- **Understanding Diseases**: Misfolded proteins are linked to diseases like Alzheimer's and Parkinson's. Predicting structures helps understand how these misfolding events occur.
- **Pathogen Research**: Predicting the structure of viral or bacterial proteins can help us to understand how pathogens harm humans and identify their vulnerabilities, aiding in the development of treatments and vaccines.
- **Genetic Disorders**: Mutations affecting protein structure can lead to genetic diseases. Predicting how these mutations alter structure helps in understanding disease mechanisms and developing therapies.

### About AlphaFold 3

AlphaFold is an AI system developed by DeepMind that predicts the 3D structure of proteins based on their amino acid sequences.
At the time of writing, AlphaFold 3 is the latest version of AlphaFold and can predict the structure of proteins, nucleic acids, and ligands.

##

When you're ready, [move on to the next page to learn how to create input files for AlphaFold 3](Input_for_AF3.md).