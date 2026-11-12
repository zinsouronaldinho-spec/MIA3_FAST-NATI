# MIA3_FAST-NATI(TP2 et Exo2)
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
POUR LE CODE DE TP2 EN C++

#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

// ================= CLASSE ETUDIANT =================
class Etudiant {
private:
    int matricule;
    string nom;

public:
    Etudiant(int m, string n) : matricule(m), nom(n) {}

    int getMatricule() const { return matricule; }
    string getNom() const { return nom; }
};

// ================= CLASSE MATIERE =================
class Matiere {
private:
    int id;
    string nom;

public:
    Matiere(int i, string n) : id(i), nom(n) {}

    int getId() const { return id; }
    string getNom() const { return nom; }
};

// ================= CLASSE ENSEIGNANT =================
class Enseignant {
private:
    int id;
    string nom;

public:
    Enseignant(int i, string n) : id(i), nom(n) {}

    int getId() const { return id; }
    string getNom() const { return nom; }
};

// ================= CLASSE SALLE =================
class Salle {
private:
    int id;
    string nom;
    int capacite;

public:
    Salle(int i, string n, int c)
        : id(i), nom(n), capacite(c) {}

    int getId() const { return id; }
    string getNom() const { return nom; }
    int getCapacite() const { return capacite; }
};

// ================= CLASSE GROUPE TD =================
class GroupeTD {
private:
    int id;
    string nom;
    vector<Etudiant*> etudiants;

public:
    GroupeTD(int i, string n) : id(i), nom(n) {}

    void ajouterEtudiant(Etudiant* e) {
        if (e && find(etudiants.begin(), etudiants.end(), e)
                    == etudiants.end()) {
            etudiants.push_back(e);
        }
    }

    int getId() const { return id; }
    string getNom() const { return nom; }

    const vector<Etudiant*>& getEtudiants() const {
        return etudiants;
    }
};

// ================= CLASSE PRESENCE =================
class Presence {
private:
    int matricule;
    bool present;

public:
    Presence(int m, bool p) : matricule(m), present(p) {}

    int getMatricule() const { return matricule; }
    bool estPresent() const { return present; }
};

// ================= CLASSE SEANCE TD =================
class SeanceTD {
private:
    int id;
    Matiere* matiere;
    GroupeTD* groupe;
    Enseignant* enseignant;
    Salle* salle;

    string date;
    string debut;
    string fin;

    vector<Presence> presences;

public:
    SeanceTD(int i, Matiere* m, GroupeTD* g,
             Enseignant* e, Salle* s,
             string d, string h1, string h2)
        : id(i), matiere(m), groupe(g),
          enseignant(e), salle(s),
          date(d), debut(h1), fin(h2) {}

    int getId() const { return id; }
    Matiere* getMatiere() const { return matiere; }
    GroupeTD* getGroupe() const { return groupe; }
    Enseignant* getEnseignant() const { return enseignant; }
    Salle* getSalle() const { return salle; }

    string getDate() const { return date; }
    string getDebut() const { return debut; }
    string getFin() const { return fin; }

    void ajouterPresence(int matricule, bool present) {
        for (auto& p : presences) {
            if (p.getMatricule() == matricule) {
                p = Presence(matricule, present);
                return;
            }
        }

        presences.emplace_back(matricule, present);
    }

    const vector<Presence>& getPresences() const {
        return presences;
    }
};

// ================= CLASSE PLATEFORME =================
class PlateformeTD {
private:
    vector<Matiere> matieres;
    vector<Etudiant> etudiants;
    vector<Enseignant> enseignants;
    vector<Salle> salles;
    vector<GroupeTD> groupes;
    vector<SeanceTD> seances;

public:
    void ajouterMatiere(int id, string nom) {
        matieres.emplace_back(id, nom);
    }

    void ajouterEtudiant(int id, string nom) {
        etudiants.emplace_back(id, nom);
    }

    void ajouterEnseignant(int id, string nom) {
        enseignants.emplace_back(id, nom);
    }

    void ajouterSalle(int id, string nom, int capacite) {
        salles.emplace_back(id, nom, capacite);
    }

    void creerGroupe(int id, string nom) {
        groupes.emplace_back(id, nom);
    }

    bool affecterEtudiant(int matricule, int idGroupe) {
        for (auto& e : etudiants) {
            if (e.getMatricule() == matricule) {
                for (auto& g : groupes) {
                    if (g.getId() == idGroupe) {
                        g.ajouterEtudiant(&e);
                        return true;
                    }
                }
            }
        }
        return false;
    }

    bool programmerSeance(int id, int idMatiere,
                          int idGroupe, int idEnseignant,
                          int idSalle, string date,
                          string debut, string fin) {

        Matiere* m = nullptr;
        GroupeTD* g = nullptr;
        Enseignant* e = nullptr;
        Salle* s = nullptr;

        for (auto& x : matieres)
            if (x.getId() == idMatiere) m = &x;

        for (auto& x : groupes)
            if (x.getId() == idGroupe) g = &x;

        for (auto& x : enseignants)
            if (x.getId() == idEnseignant) e = &x;

        for (auto& x : salles)
            if (x.getId() == idSalle) s = &x;

        if (!m || !g || !e || !s || debut >= fin) {
            cout << "Informations invalides.\n";
            return false;
        }

        if (g->getEtudiants().size() >
            static_cast<size_t>(s->getCapacite())) {
            cout << "Capacite de salle insuffisante.\n";
            return false;
        }

        for (const auto& se : seances) {
            bool memeDate = se.getDate() == date;
            bool chevauchement =
                debut < se.getFin() && se.getDebut() < fin;

            if (memeDate && chevauchement) {
                if (se.getSalle()->getId() == idSalle) {
                    cout << "Conflit de salle !\n";
                    return false;
                }

                if (se.getEnseignant()->getId() == idEnseignant) {
                    cout << "Conflit d'enseignant !\n";
                    return false;
                }

                if (se.getGroupe()->getId() == idGroupe) {
                    cout << "Groupe deja occupe !\n";
                    return false;
                }
            }
        }

        seances.emplace_back(id, m, g, e, s,
                             date, debut, fin);

        cout << "Seance programmee avec succes !\n";
        return true;
    }

    bool enregistrerPresence(int idSeance,
                             int matricule, bool present) {
        for (auto& s : seances) {
            if (s.getId() == idSeance) {
                for (auto* e : s.getGroupe()->getEtudiants()) {
                    if (e->getMatricule() == matricule) {
                        s.ajouterPresence(matricule, present);
                        return true;
                    }
                }
            }
        }
        return false;
    }

    void afficherCalendrier() const {
        cout << "\n===== CALENDRIER DES TD =====\n";

        for (const auto& s : seances) {
            cout << s.getDate() << " | "
                 << s.getDebut() << " - " << s.getFin()
                 << " | " << s.getMatiere()->getNom()
                 << " | " << s.getGroupe()->getNom()
                 << " | " << s.getSalle()->getNom()
                 << " | " << s.getEnseignant()->getNom()
                 << endl;
        }
    }

    void afficherBilan() const {
        cout << "\n===== BILAN DES PRESENCES =====\n";

        for (const auto& s : seances) {
            int presents = 0;
            int absents = 0;

            for (const auto& p : s.getPresences()) {
                if (p.estPresent())
                    presents++;
                else
                    absents++;
            }

            cout << s.getMatiere()->getNom()
                 << " - " << s.getGroupe()->getNom()
                 << "\nPresents : " << presents
                 << "\nAbsents : " << absents << endl;
        }
    }
};

// ================= PROGRAMME PRINCIPAL =================
int main() {
    PlateformeTD plateforme;

    plateforme.ajouterMatiere(1, "Mathematiques");
    plateforme.ajouterMatiere(2, "Informatique");

    plateforme.ajouterEtudiant(101, "Amina");
    plateforme.ajouterEtudiant(102, "Paul");
    plateforme.ajouterEtudiant(103, "Grace");

    plateforme.ajouterEnseignant(1, "M. Kossi");
    plateforme.ajouterEnseignant(2, "Mme Dossou");

    plateforme.ajouterSalle(1, "Salle A12", 40);
    plateforme.ajouterSalle(2, "Salle B05", 25);

    plateforme.creerGroupe(1, "MI1");
    plateforme.creerGroupe(2, "MI2");

    plateforme.affecterEtudiant(101, 1);
    plateforme.affecterEtudiant(102, 1);
    plateforme.affecterEtudiant(103, 2);

    plateforme.programmerSeance(
        1, 1, 1, 1, 1,
        "2026-11-16", "08:00", "10:00"
    );

    plateforme.programmerSeance(
        2, 2, 2, 2, 2,
        "2026-11-16", "10:00", "12:00"
    );

    plateforme.enregistrerPresence(1, 101, true);
    plateforme.enregistrerPresence(1, 102, false);
    plateforme.enregistrerPresence(2, 103, true);

    plateforme.afficherCalendrier();
    plateforme.afficherBilan();

    return 0;
}
POUR LE CODE DE DIAGRAMME EN UML

@startuml
left to right direction

actor Administrateur
actor Enseignant
actor Etudiant

rectangle "Plateforme de gestion des TD" {

 usecase "Enregistrer les matieres" as UC1
 usecase "Creer les groupes TD" as UC2
 usecase "Affecter les etudiants" as UC3
 usecase "Programmer une seance" as UC4
 usecase "Verifier les conflits" as UC5
 usecase "Consulter le calendrier" as UC6
 usecase "Enregistrer les presences" as UC7
 usecase "Produire le bilan" as UC8

}

Administrateur --> UC1
Administrateur --> UC2
Administrateur --> UC3
Administrateur --> UC4
Administrateur --> UC8

Enseignant --> UC4
Enseignant --> UC6
Enseignant --> UC7

Etudiant --> UC6

UC4 .> UC5 : <<include>>

@enduml
