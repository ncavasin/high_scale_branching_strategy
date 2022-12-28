# Branching strategy with semantic versioning
We propose the following three environments to achieve a fully-automated CICD flow:
- **develop**: feature integration environment.
- **release**: testing environment.
- **master**: productive environment.

On top of them, we propose these temporal branches which will be merged to their proper environment:
- **feature/\*\***: to develop new features branched from release environment (less bug-prone).
- **bugfix/\*\***: to fix bugs found on release environment.
- **hotfix/\*\***: to fix bugs found on productive environment.
---
### Behavior
##### Development environment
**Stage 1:** integrate new feature to development environment.
Create a Pull-Request from temporal branch ``feature/XYZ`` against ``develop`` environment to trigger its pipeline.
- On pipe success approve and merge Pull-Request withou any code-review nor approval (speed-up integration process).
- On pipe failure fix errors/bugs and restart process.

**Stage 2**: promote feature to release environment.    
Create a Pull-Request from temporal branch ``feature/XYZ`` against ``release`` environment to trigger its pipeline.
- On pipe success, assign a reviewer and wait for him to approve and merge the new feature.
    - ``release`` environment semver tagging will be automated.
- On pipe failure, fix error/bugs and restart process from **Stage 1**.

##### Release environment

1. Opción A: pasaje de bugfix/** a release/stable vía PR para disparar el pipeline de release/stable.
    - Si el pipeline se ejecuta exitosamente se requiere Code Review + approval para mergear.
        - Se automatiza el tagging del ambiente de release/stable.
        - Se automatiza el merge del bugfix a Develop.
    - Si el pipeline falla en alguno de sus stages, no se permite el merge a release/stable (restricción desde el repo) porque se debe ajustar código y se reinicia el proceso.

2. Opción B: pasaje de release/stable a master vía PR para disparar el pipe de master.
    - Si el pipeline se ejecuta exitosamente se requiere Code Review + approval para mergear.
        - Se automatiza el tagging del ambiente de Producción.
        - Si el pipeline falla en alguno de sus stages = ajustar código y reiniciar proceso.

> Nota aclaratoria: para el caso de ramas release/**, se podrá manejar "más de una rama" que permita disponibilizar diferentes release candidates. No obstante, siempre existirá un ambiente de release estable, llamado release/stable, sobre el cuál se permite realizar QA y al cuál el merge_handler apuntará al momento de realizar un merge.

##### Productive environment
1. Pasaje de hotfix/** a master vía PR para disparar el pipeline de master.
    - Si el pipeline se ejecuta exitosamente se requiere Code Review + approval para mergear.
    - Se automatiza el tagging del ambiente de Producción.
     - Se automatiza el merge del hotfix a Release y a Develop.
     - Si el pipeline falla en alguno de sus stages, no se permite el merge a Master (restricción desde el repo) porque se debe ajustar código y reiniciar el proceso.

Todos los pipelines agregarán un reporte a modo comentario en la PR correspondiente para indicar el resultado de cada stage atravesado.

### Protección de ramas y Pull Requests:
- No se permite commitear ni mergear a las ramas base por definición (develop, release, master).
- No se permite el merge a ningún ambiente si el pipeline no finalizó con un resultado exitoso.
 - Todos los pasos a dichas ramas se realizan a través de Pull-Requests:
    - Si hay merge conflicts se denotan en el PR hacia la rama destino.
    - Si falla el PR hacia destino, agregar comentario del error/es e la ejecución en el PR (comment PR). Ajustar a API Bitbucket tomando error de pipeline (get exit status → api bb). Armar un script en python que actualice PR.
- Se debe realizar aprobación de la siguiente manera:
    - Develop de forma automática si pasa la revisión del pipeline (merge automático)
    - Release y Master -> se ejecuta pipeline y se debe aprobar "manual". (merge manual, en base a un analista funcional [persona humana])



