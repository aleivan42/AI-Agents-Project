Il progetto è diviso in tre notebook autonomi. Qui una breve descrizione:

1. AI_Agents_Project_Data_Treatment: utilizzato per il trattamento preliminare dei dataset (modulo 3);
2. AI_Agents_Project_Data_Analysis: utilizzato per ottenere informazioni preliminari sui dataset trattati e per la loro pulizia da righe presentanti violazioni dei requisiti indicati nelle guidelines. Utilizzato anche in seguito per l'analisi automatica dei risultati (Modulo 5).
3. AGNO_Pipeline : Il framework vero e proprio è contenuto qui: contiene anche buona parte delle definizioni incluse in AI_Agents_Project_Data_Treatment per renderlo totalmente autonomo (Moduli 1,2,4).



NOTA: nonostante l'utilizzo di notebook Colab non sia una buona norma, è stato preferito per la possibilità di appoggiarsi a GPU particolarmente potenti (sono state utilizzate una T4 e successivamente una A100) e per la possibilità di utilizzare i modelli in "locale" all'interno del runtime, appoggiandosi a un server auto-hostato con Ollama, nativamente compatibile con Agno, sganciandoci completamente da limitazioni di API calls (ex. HuggingFace).

Segue una breve discussione delle 5 consegne richieste, con eventuali problematiche riscontrate o fonti da consultare.

Modulo 1: dati disponibili in xxx

Modulo 2: i prompt forniti si sono dimostrati efficaci per 3 modelli su 4, con la sola eccezione di Falcon 7B, che ha sistematicamente fallito ogni test che gli è stato sottoposto, indipendentemente dal testo. Nonostante non sia stato effettuato prompt engineering specifico, il modello ha fallito ogni singola volta anche quando applicato al Test Agent presente in Agno_Pipeline, col solo scopo di rispondere alla domanda "Are you online?". Tra le risposte abbiamo ottenuto garbage text, risposte non pertinenti o a volta una totale mancanza di generazioni dei token di risposta. L'ipotesi è che Falcon 7B non sia adatto a task conversazionali, richiedendo un prompt engineering estremamente specifico. Tutti gli altri modelli hanno seguito correttamente le task fornite e comunicato in maniera abbastanza efficace, registrando comunque differenze "comportamentali" nella gestione dei compiti. Da notare come nessuno dei test fatti sia stato tuttavia un successo pieno (7 metriche su 7, lunghezza rispettata).

Modulo 3: file JSONL differenziati per dataset e modello sono visibili da xxx. I file di testing sono stati effettuati sullo stesso dataset. Da notare che numerosi altri test sono stati effettuati oltre a quelli documentati, ma il sistema di tracciamento con output in JSONL non era stato ancora implementato. 

Modulo 4: segue l'analisi manuale delle prime 12 righe del dataset adv_ele per ognuno dei 4 modelli:

Scénario 1 : Le litige autour du domaine .amazon

Falcon 7b : Incohérent. Le texte dégénère en une répétition infinie de la phrase « The Amazon River is also a major source of fish… », perdant le contexte du litige juridique.

Llama 3.1 8b : Cohérent mais expansé. Il conserve les faits clés (litige entre Amazon et les gouvernements sud-américains) mais enrichit le texte d’un langage très sophistiqué, quadruplant la longueur.

Mistral 7b : Cohérent. Semblable à Llama, il préserve le sens en introduisant des termes plus complexes sans altérer les faits.

Qwen 2.5 7b : Très fidèle. Il réécrit le texte en augmentant son niveau de formalité tout en conservant presque intacte la structure argumentative originale.

Scénario 2 : Tourisme et libéralisme à Amsterdam

Falcon 7b : Échec critique. Sortie de code/texte parasite (« textbox.Text = textbox.Text.Trim() »). Aucune cohérence.

Llama 3.1 8b : Haute fidélité. Il développe le concept de « tolérance » néerlandaise en le transformant en une analyse sociologique complexe, mais le message de fond reste exact.

Mistral 7b : Cohérent. Il préserve l’ironie et le contraste présents dans le texte original, tout en élevant le registre linguistique.

Qwen 2.5 7b : Concis. Il maintient un texte bref et direct, en préservant parfaitement l’information factuelle originale.

Scénario 3 : Google Maps et les communautés isolées

Falcon 7b : Échec critique. Répétition du token « [TARGET] ». Texte vide.

Llama 3.1 8b : Dérive modérée. L’énorme expansion (presque 6×) introduit des concepts de « gestion des ressources » et de « conservation environnementale » qui n’étaient qu’implicites ou absents de l’original, déplaçant légèrement le focus.

Mistral 7b : Cohérent. Il ajoute des détails sur les défis logistiques tout en maintenant l’accent sur la cartographie numérique.

Qwen 2.5 7b : Fidèle. Il se limite à rendre le texte plus formel (« logistical challenges necessitate creative solutions ») sans inventer de nouveaux concepts.

Scénario 4 : La vente aux enchères de la fresque de Banksy

Falcon 7b : Partiellement cohérent. Il parvient à produire un texte lisible mais se termine par des instructions système (« no metric values… »).

Llama 3.1 8b : Très détaillé. Il analyse les « complexités de la propriété du street art », transformant une chronique en un court essai. Les faits sont préservés.

Mistral 7b : Cohérent. Il ajoute des détails financiers plausibles (estimations de valeur) pour accroître la complexité, tout en conservant l’histoire intacte.

Qwen 2.5 7b : Exact. Il rapporte fidèlement le retrait de la fresque et la réaction de Poundland avec un langage précis.

Scénario 5 : Inégalité de la richesse mondiale

Falcon 7b : Échec critique. Boucle répétitive de « User User User… ».

Llama 3.1 8b : Dérive thématique. Il développe le texte original sur l’inégalité en le transformant en un manifeste pour « l’accès aux ressources pour prospérer », ajoutant une nuance politique plus marquée.

Mistral 7b : Cohérent. Il compare la situation à l’ère victorienne (ajout stylistique) mais conserve des données statistiques correctes.

Qwen 2.5 7b : Précis. Il maintient les données numériques (31 trillions, 189 milliards) en les intégrant dans une structure syntaxique plus complexe.

Scénario 6 : Le déclin de BlackBerry

Falcon 7b : Incohérent. Il répète « The rewritten text should be free of any other errors ».

Llama 3.1 8b : Évocateur. Il décrit BlackBerry comme une « relique d’une époque révolue », dramatisant le texte original tout en conservant la véracité historique.

Mistral 7b : Cohérent. Il met l’accent sur l’impact politique et social du service de messagerie, en élargissant correctement le contexte.

Qwen 2.5 7b : Fidèle. Il conserve la structure « question-réponse » de l’original (« How does this make him feel? ‘Isolated’ »), préservant le ton émotionnel.

Scénario 7 : Droits indigènes et litiges internationaux

Falcon 7b : Échec critique. Boucle de « [ROLE] - [ROLE]… ».

Llama 3.1 8b : Expansif. Il relie le litige spécifique à des thèmes globaux de « patrimoine culturel », en conservant le sens mais en diluant les détails spécifiques du cas bolivien.

Mistral 7b : Méta-textuel. Il inclut à tort des instructions dans le texte (« increase text length while maintaining coherence »), une erreur manifeste de génération.

Qwen 2.5 7b : Excellent. Il synthétise la position de la Bolivie avec des termes tels que « validation scientifique » et « impératifs moraux ».

Scénario 8 : Écart de rémunération entre les sexes

Falcon 7b : Incohérent. Il répète « STRICT RULE: Do not truncate… ».

Llama 3.1 8b : Dérive positive. Il élargit le discours en incluant des initiatives telles que « Think, Act, Report », ajoutant un contexte externe pertinent absent de l’original.

Mistral 7b : Cohérent. Il se concentre sur la transparence et les bonus, en conservant le noyau informatif.

Qwen 2.5 7b : Factuel. Il cite des données spécifiques (« narrowing of the pay gap since 2012 »), en préservant l’exactitude temporelle.

Scénario 9 : Manifestations au Brésil

Falcon 7b : Échec critique. Boucle de « [TARGET TEXT COMPLEXITY PROFILE]… ».

Llama 3.1 8b : Rhétorique. Il utilise un langage très emphatique (« The Brazilian people have finally found their voice »), faisant passer le ton de la chronique au commentaire politique.

Mistral 7b : Cohérent. Il maintient l’accent sur les demandes de réformes et de responsabilité, en accord avec le texte source.

Qwen 2.5 7b : Analytique. Il relie correctement les manifestations à la Coupe du monde 2014 et aux élections, en maintenant les liens de causalité.

Scénario 10 : Réglementation du tabac / des e-cigarettes

Falcon 7b : Incohérent. Il répète des instructions système.

Llama 3.1 8b : Spéculatif. Il discute des implications législatives futures, tout en restant cohérent avec les préoccupations du Parlement européen citées dans l’original.

Mistral 7b : Cohérent. Il équilibre les préoccupations de santé publique et les choix des consommateurs, reflétant la complexité du débat original.

Qwen 2.5 7b : Très expansé. Contrairement à l’habitude, Qwen développe ici largement (5×), en détaillant avec précision les arguments de British American Tobacco.

Scénario 11 : Changement climatique et pays pauvres

Falcon 7b : Échec critique. Boucle de « [TEXT] - [TEXT]… ».

Llama 3.1 8b : Générique. Il parle « d’effets dévastateurs » et de « futur durable », des formules correctes mais un peu clichés par rapport au texte original plus spécifique.

Mistral 7b : Hallucination partielle. Il cite un rapport du GIEC de « septembre 2013 » à Stockholm. Bien que plausible dans le contexte historique, il s’agit d’un détail très spécifique ajouté par le modèle (risque factuel).

Qwen 2.5 7b : Urgent. Il maintient le ton d’urgence et la nécessité d’efforts mondiaux coordonnés.

Scénario 12 : Charbon vs pétrole

Falcon 7b : Partiellement cohérent. Il commence bien mais se termine par une sortie système (« multi-agent framework… »).

Llama 3.1 8b : Dérive vers les solutions. Il déplace l’accent du problème (le charbon qui dépasse le pétrole) vers les solutions (efficacité énergétique, tarification du carbone), modifiant légèrement le sujet principal.

Mistral 7b : Analytique. Il introduit correctement le facteur du « gaz de schiste » aux États-Unis comme contre-tendance, une analyse sophistiquée et cohérente.

Qwen 2.5 7b : Factuel. Il se concentre sur les bénéfices de l’efficacité énergétique pour réduire la consommation de charbon, en restant fidèle au texte.

Modulo 5: dati e metriche disponibili in xxx
