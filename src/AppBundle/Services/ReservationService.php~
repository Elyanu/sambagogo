<?php

namespace AppBundle\Services;

use AppBundle\Entity\Commande;
use Doctrine\ORM\EntityManager;
use AppBundle\Entity\Billet;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class ReservationService extends Controller
{
    protected $em;
    protected $gratuit;
    protected $enfant;
    protected $reduit;
    protected $normal;
    protected $senior;

    public function __construct(EntityManager $em, $gratuit, $enfant, $reduit, $normal, $senior)
    {
        $this->reduit = $reduit;
        $this->normal = $normal;
        

    }

    // ÉTAPE 1 : Instancie une nouvelle commande à partir des infos du premier formulaire
    public function createCommande($session, $email)
    {
        $today = new \DateTime('today');
        $hour = date('H');
        
        $commande = new Commande();
        $commande->setSession($session);
        $commande->setEmail($email);
        $this->saveCommande($commande);
        $this->setToken($commande->getId());
        return $commande;
    }

    // Persiste la commande en base de données
    public function saveCommande($commande)
    {
        $this->em->persist($commande);
        $this->em->flush();
    }

    // Attribue un numéro de commande unique (date + id, au format 160912_15)
    public function setToken($id)
    {
        $commande = $this->getCommande($id);
        $date = date('ymd');
        $commande->setToken($date.'_'.$id);
        $this->saveCommande($commande);
    }

    // Récupère une commande par son id
    public function getCommande($id)
    {
        $repository = $this->em->getRepository('AppBundle:Commande');
        return $repository->find($id);
    }

    // ÉTAPE 2 : Instancie un billet pour chaque formulaire rempli
    public function createBillet($id, $prenom, $nomf, $dateNaissance, $tarifReduit)
    {
        $billet = new Billet();
        $billet->setCommande($this->getCommande($id));
        
        $billet->setPrenom($prenom);
        $billet->setNomf($nomf);
        
        $billet->setDateNaissance($dateNaissance);
        $billet->setTarifReduit($tarifReduit);
        $this->saveBillet($billet);
        $billetId = $billet->getId();
        $this->getAgeVisiteur($billetId);
        $this->getPrix($billetId);
    }

    // Persiste le billet en base de données
    public function saveBillet($billet)
    {
        $this->em->persist($billet);
        $this->em->flush();
    }

    // Récupère un billet par son id
    public function getBillet($billetId)
    {
        $repository = $this->em->getRepository('AppBundle:Billet');
        return $repository->find($billetId);
    }

    // Supprime un billet de la base de données
    public function removeBillet($billetId)
    {
        $billet = $this->getBillet($billetId);
        $this->em->remove($billet);
        $this->em->flush();
    }

    // Calcule la différence entre la date de la visite et la date de naissance saisie
    // Persiste l'âge du visiteur en base de données
    public function getAgeVisiteur($billetId)
    {
        $billet = $this->getBillet($billetId);
        $dateNaissance = $billet->getDateNaissance();
        $visite = $billet->getDateVisite();
        $interval = $visite->diff($dateNaissance);
        $age = $interval->y;
        $billet->setAge($age);
        $this->saveBillet($billet);
    }

    // Récupère l'âge du visiteur et attribue le tarif
    // Persiste le prix du billet en base de données
    public function getPrix($billetId)
    {
        $billet = $this->getBillet($billetId);
        $age = $billet->getAge();
        $id = $billet->getCommande();
        $tarifReduit = $billet->isTarifReduit();
        $commande = $this->getCommande($id);
        

         if (!$tarifReduit) {
                
                    $billet->setPrix($this->normal);
                }
                else {
                    $billet->setPrix($this->normal/2);
                }
            
        $this->saveBillet($billet);
    }

    // Récupère le tarif de tous les billets de la commande
    // Aditionne et persiste le montant en base de données
    public function getMontant($id)
    {
        $commande = $this->getCommande($id);
        $montant = 0;
        foreach ($commande->getBillets() as $billet) {
            $montant += $billet->getPrix();
        }
        $commande->setMontant($montant);
        $this->saveCommande($commande);
        return $montant;
    }

  

   

    // Retourne le nombre de billets d'une commande donnée
    public function billetsCommande($id)
    {
        $commande = $this->getCommande($id);
        $quantite = 0;
        foreach ($commande->getBillets() as $billet) {
            $quantite++;
        }
        return $quantite;
    }

    // Marque les billets comme réservés une fois le paiement effectué
    public function changeStatutTransaction($id)
    {
        $commande = $this->getCommande($id);
        foreach ($commande->getBillets() as $billet)
        {
            $billet->setStatutTransaction(true);
            $this->saveBillet($billet);
        }
    }

    public function getSession($id)
    {
        $commande = $this->getCommande($id);
        return $commande->getSession();
    }
}
