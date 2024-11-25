# Arquitectura Escalable per a WordPress

En aquesta pràctica, dissenyarem i implementarem una arquitectura escalable i segura per a una aplicació WordPress, utilitzant els serveis d’AWS. Aquesta arquitectura constarà dels següents components:

- **Balancejador de Càrrega (Load Balancer)**: Serà responsable de distribuir uniformement el tràfic entrant entre múltiples instàncies EC2. Això assegura una alta disponibilitat i resiliència davant augments de càrrega.

- **Instàncies EC2**: Aquestes màquines virtuals allotjaran l’aplicació WordPress. Estaran dins d’un grup d’autoscaling, que permetrà ajustar dinàmicament el nombre d’instàncies en funció de la demanda.

- **Base de Dades RDS**: Utilitzarem Amazon RDS per gestionar la base de dades de WordPress.

## Tasques

1. [Creació de la VPC i les subxarxes](./vpc-networking.md)
2. [Taules d'Enrutament i Associacions](./routing.md)
3. [Grups de Seguretat](./security.md)
4. [Creació de la Base de Dades RDS](./rds.md)
5. [Creació de l'instància EC2](./ec2.md)
6. [Wordpress](./wordpress.md)
