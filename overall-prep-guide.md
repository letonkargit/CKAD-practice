Yet to fill in the details

During Exam:
1. set vim settings
    $ vim ~/.vimrc
    set et ai
    set sw=2 ts=2 sts=2
2. set below aliases: they already set k as kubectl
alias kg="k get"
alias kd="k describe"
alias kaf="k apply -f"
alias krm="k delete --grace-period=0 --force"
alias kn="k config set-context --current --namespace"
export dry="--dry-run=client -o yaml"


--Check allowed resources
https://kubernetes.io/docs
https://kubernetes.io/blog
https://helm.sh/docs
