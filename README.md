# MIA3_FAST-NATI
Pour exo 2 , voilà son code d'implantation en c++

#include <iostream>
#include <vector>
#include <string>
#include <fstream>
#include <iomanip>
#include <limits>

using namespace std;

// ===============================
// CLASSE MATIERE
// ===============================

class Matiere {
private:
    string code;
    string nom;
    double coefficient;

public:
    Matiere(string c, string n, double coef)
        : code(c), nom(n), coefficient(coef) {}

    string getCode() const {
        return code;
    }

    string getNom() const {
        return nom;
    }

    double getCoefficient() const {
        return coefficient;
    }
};

// ===============================
// CLASSE NOTE
// ===============================

class Note {
private:
    string codeMatiere;
    double valeur;

public:
    Note(string code, double v)
        : codeMatiere(code), valeur(v) {}

    string getCodeMatiere() const {
        return codeMatiere;
    }

    double getValeur() const {
        return valeur;
    }
};

// ===============================
// CLASSE UE
// ===============================

class UE {
private:
    string code;
    string nom;
    int credit;
    vector<Matiere> matieres;

public:
    UE(string c, string n, int cr)
        : code(c), nom(n), credit(cr) {}

    void ajouterMatiere(Matiere m) {
        matieres.push_back(m);
    }

    string getNom() const {
        return nom;
    }

    int getCredit() const {
        return credit;
    }

    const vector<Matiere>& getMatieres() const {
        return matieres;
    }

    double calculerMoyenneUE(
        const vector<Note>& notes) const {

        double somme = 0;
        double totalCoef = 0;

        for (const auto& m : matieres) {
            for (const auto& n : notes) {

                if (m.getCode() == n.getCodeMatiere()) {

                    somme += n.getValeur()
                             * m.getCoefficient();

                    totalCoef += m.getCoefficient();
                    break;
                }
            }
        }

        if (totalCoef == 0)
            return 0;

        return somme / totalCoef;
    }

    bool estValidee(const vector<Note>& notes) const {
        return calculerMoyenneUE(notes) >= 10;
    }
};

// ===============================
// CLASSE FILIERE
// ===============================

class Filiere {
private:
    string code;
    string nom;
    vector<UE> ues;

public:
    Filiere(string c, string n)
        : code(c), nom(n) {}

    void ajouterUE(UE ue) {
        ues.push_back(ue);
    }

    string getNom() const {
        return nom;
    }

    const vector<UE>& getUEs() const {
        return ues;
    }
};

// ===============================
// CLASSE ETUDIANT
// ===============================

class Etudiant {
private:
    string matricule;
    string nom;
    string niveau;
    const Filiere* filiere;
    vector<Note> notes;

public:
    Etudiant(string mat, string n, string niv,
             const Filiere* f)
        : matricule(mat), nom(n),
          niveau(niv), filiere(f) {}

    void ajouterNote(Note note) {
        notes.push_back(note);
    }

    double calculerMoyenne() const {

        double somme = 0;
        double totalCredits = 0;

        for (const auto& ue : filiere->getUEs()) {

            double moyenneUE =
                ue.calculerMoyenneUE(notes);

            somme += moyenneUE * ue.getCredit();

            totalCredits += ue.getCredit();
        }

        if (totalCredits == 0)
            return 0;

        return somme / totalCredits;
    }

    void afficherReleve() const {

        cout << "\n====================================\n";
        cout << "       RELEVE DE NOTES\n";
        cout << "====================================\n";

        cout << "Nom : " << nom << endl;
        cout << "Matricule : " << matricule << endl;
        cout << "Niveau : " << niveau << endl;
        cout << "Filiere : " << filiere->getNom() << endl;

        cout << "\nDETAIL DES UE\n";

        for (const auto& ue : filiere->getUEs()) {

            double moyenne =
                ue.calculerMoyenneUE(notes);

            cout << "\nUE : " << ue.getNom() << endl;

            for (const auto& m : ue.getMatieres()) {

                for (const auto& n : notes) {

                    if (m.getCode() == n.getCodeMatiere()) {
                        cout << "  " << m.getNom()
                             << " : " << n.getValeur()
                             << "/20" << endl;
                        break;
                    }
                }
            }

            cout << "Moyenne UE : "
                 << fixed << setprecision(2)
                 << moyenne << "/20" << endl;

            cout << "Credits : " << ue.getCredit() << endl;

            cout << "Statut : "
                 << (ue.estValidee(notes)
                     ? "VALIDEE" : "NON VALIDEE")
                 << endl;
        }

        cout << "\n====================================\n";

        cout << "MOYENNE GENERALE : "
             << fixed << setprecision(2)
             << calculerMoyenne() << "/20" << endl;

        cout << "====================================\n";
    }

    // Generation du graphique HTML/SVG

    void genererGraphique() const {

        ofstream fichier("resultats.html");

        if (!fichier) {
            cout << "Erreur de creation du graphique.\n";
            return;
        }

        fichier << R"(
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Graphique des resultats</title>
<style>
body {
    font-family: Arial;
    background: #f2f5f9;
    text-align: center;
    padding: 25px;
}
h1 { color: #17365d; }
svg {
    background: white;
    max-width: 850px;
    width: 95%;
    border-radius: 12px;
}
</style>
</head>
<body>
<h1>Releve graphique des resultats</h1>
<h2>)" << nom << R"(</h2>
<svg viewBox="0 0 850 480"
     xmlns="http://www.w3.org/2000/svg">
)";

        // Axes du graphique

        fichier << R"(
<line x1="90" y1="390" x2="800" y2="390"
      stroke="black" stroke-width="2"/>
<line x1="90" y1="50" x2="90" y2="390"
      stroke="black" stroke-width="2"/>
)";

        // Graduations de 0 a 20

        for (int i = 0; i <= 20; i += 5) {

            int y = 390 - i * 15;

            fichier << "<text x=\"65\" y=\"" << y + 5
                    << "\" font-size=\"13\">"
                    << i << "</text>\n";

            fichier << "<line x1=\"90\" y1=\"" << y
                    << "\" x2=\"800\" y2=\"" << y
                    << "\" stroke=\"#dddddd\"/>\n";
        }

        int x = 130;

        for (const auto& ue : filiere->getUEs()) {

            double moyenne = ue.calculerMoyenneUE(notes);

            int hauteur = static_cast<int>(moyenne * 15);
            int y = 390 - hauteur;

            fichier << "<rect x=\"" << x
                    << "\" y=\"" << y
                    << "\" width=\"75\" height=\"" << hauteur
                    << "\" fill=\""
                    << (moyenne >= 10 ? "#2196F3" : "#E74C3C")
                    << "\"/>\n";

            fichier << "<text x=\"" << x + 37
                    << "\" y=\"" << y - 10
                    << "\" text-anchor=\"middle\""
                    << " font-size=\"14\">"
                    << fixed << setprecision(2)
                    << moyenne << "</text>\n";

            fichier << "<text x=\"" << x + 37
                    << "\" y=\"415\" text-anchor=\"middle\""
                    << " font-size=\"12\">"
                    << ue.getNom() << "</text>\n";

            x += 150;
        }

        fichier << R"(
</svg>
<h2>Moyenne generale : )"
                << fixed << setprecision(2)
                << calculerMoyenne()
                << R"(/20</h2>
<p>Bleu : UE validee | Rouge : UE non validee</p>
</body>
</html>
)";

        fichier.close();

        cout << "\nGraphique genere : resultats.html\n";
    }
};

// ===============================
// PROGRAMME PRINCIPAL
// ===============================

int main() {

    // Creation des UE

    UE ue1("UE01", "Mathematiques", 6);

    ue1.ajouterMatiere(
        Matiere("M01", "Analyse", 3));

    ue1.ajouterMatiere(
        Matiere("M02", "Algebre", 2));

    ue1.ajouterMatiere(
        Matiere("M03", "Probabilites", 1));


    UE ue2("UE02", "Informatique", 4);

    ue2.ajouterMatiere(
        Matiere("M04", "Algorithmique", 2));

    ue2.ajouterMatiere(
        Matiere("M05", "Programmation C++", 2));


    UE ue3("UE03", "Physique", 2);

    ue3.ajouterMatiere(
        Matiere("M06", "Mecanique", 1));

    ue3.ajouterMatiere(
        Matiere("M07", "Electricite", 1));


    // Creation de la filiere

    Filiere filiere("MI", "Mathematiques et Informatique");

    filiere.ajouterUE(ue1);
    filiere.ajouterUE(ue2);
    filiere.ajouterUE(ue3);


    // Creation de l'etudiant

    Etudiant etudiant("MI2026-001", "Jean", "L3",
                      &filiere);


    // Attribution des notes

    etudiant.ajouterNote(Note("M01", 14));
    etudiant.ajouterNote(Note("M02", 12));
    etudiant.ajouterNote(Note("M03", 16));

    etudiant.ajouterNote(Note("M04", 15));
    etudiant.ajouterNote(Note("M05", 13));

    etudiant.ajouterNote(Note("M06", 11));
    etudiant.ajouterNote(Note("M07", 9));


    // Affichage du releve

    etudiant.afficherReleve();


    // Generation du graphique

    etudiant.genererGraphique();

    return 0;
}
POUR L'EXO 2 ,LE CODE DE UML

@startuml
title Gestion des étudiants et des résultats académiques

skinparam classAttributeIconSize 0
skinparam linetype ortho

class Etudiant {
  - matricule : String
  - nom : String
  - niveau : String
  - filiere : Filiere*
  - notes : vector<Note>
  + ajouterNote(note : Note) : void
  + calculerMoyenne() : double
  + afficherReleve() : void
  + genererGraphique() : void
}

class Filiere {
  - code : String
  - nom : String
  - ues : vector<UE>
  + ajouterUE(ue : UE) : void
}

class UE {
  - code : String
  - nom : String
  - credit : int
  - matieres : vector<Matiere>
  + ajouterMatiere(m : Matiere) : void
  + calculerMoyenneUE() : double
  + estValidee() : bool
}

class Matiere {
  - code : String
  - nom : String
  - coefficient : double
  + getCode() : String
  + getNom() : String
  + getCoefficient() : double
}

class Note {
  - codeMatiere : String
  - valeur : double
  + getCodeMatiere() : String
  + getValeur() : double
}

Filiere "1" *-- "1..*" UE : contient
UE "1" *-- "1..*" Matiere : compose
Filiere "1" -- "0..*" Etudiant : accueille
Etudiant "1" *-- "0..*" Note : possede
Note "*" --> "1" Matiere : concerne

@enduml
