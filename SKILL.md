---
name: copydesk-padronizacao-dialogos
description: >
  Use esta skill quando precisar realizar auditoria e refinamento editorial profunda em manuscritos.
  Ativa quando o usuário pede para revisar diálogos, padronizar pontuação, converter aspas retas em tipográficas, validar convenções de estilo ou gerar relatórios editoriais.
  Entrega texto com acabamento invisível, padronização tipográfica perfeita e relatório detalhado de intervenções realizadas.
---
# Skill: Copydesk e Padronização de Diálogos

## Propósito
Realizar auditoria e refinamento editorial de alta precisão em manuscritos, atuando como etapa final de polimento. A skill não faz revisão ortográfica superficial — realiza varredura profunda e estrutural para garantir unidade rítmica, visual e gramatical, com padronização tipográfica absoluta e validação completa de diálogos.

## O que esta skill NÃO faz
- Não reescreve conteúdo — apenas padroniza formatação
- Não interpreta significado — apenas valida consistência mecânica
- Não corrige erros conceituais — apenas erros tipográficos e de estilo
- Não substitui revisor humano — complementa o processo editorial

## Fontes
- Referência: `skill externa de Copydesk e Padronização de Diálogos.pdf` (skill externa EscreveAI)
- Internet: Manual de Estilo APUE (Associação Brasileira de Editores de Universidades)
- Internet: Normas da ABNT para publicações (NBR 14724)
- Internet: Guia de Estilo da Folha de S.Paulo
- Internet: Manual de Redação da Folha de S.Paulo (edição mais recente)
- Internet: Chicago Manual of Style (17ª edição)

## Pré-requisitos
- Python 3.10+
- `re` (biblioteca padrão) — para expressões regulares
- `unicodedata` (biblioteca padrão) — para normalização Unicode
- `dataclasses` (biblioteca padrão) — para estruturas de dados
- `typing` (biblioteca padrão) — para type hints
- `pathlib` (biblioteca padrão) — para manipulação de caminhos

## Arquitetura de Dados

### Estrutura de Configuração
```python
from dataclasses import dataclass, field
from typing import List, Dict, Tuple, Optional, Set
from enum import Enum
import re
from pathlib import Path

class ConvencaoDialogo(Enum):
    TRAVESSOES = "travessoes"
    ASPAS = "aspas"
    MISTA = "mista"

class ConvencaoPontuacao(Enum):
    PORTUGUES_BRASIL = "pt_BR"
    PORTUGUES_PORTUGAL = "pt_PT"
    INGLES_AMERICANO = "en_US"
    INGLES_BRITANICO = "en_GB"
    ESPANHOL = "es"

class NivelAuditoria(Enum):
    BASICA = "basica"  # Apenas erros óbvios
    PADRAO = "padrao"  # Padronização completa
    PROFUNDA = "profunda"  # Análise rítmica e contextual

@dataclass
class ConfiguracaoEditorial:
    """Configurações do guia de estilo."""
    convencao_dialogo: ConvencaoDialogo = ConvencaoDialogo.TRAVESSOES
    convencao_pontuacao: ConvencaoPontuacao = ConvencaoPontuacao.PORTUGUES_BRASIL
    nivel_auditoria: NivelAuditoria = NivelAuditoria.PADRAO
    
    # Aspas tipográficas
    aspas_abertura: str = "\u201c"  # "
    aspas_fechamento: str = "\u201d"  # "
    aspas_simples_abertura: str = "\u2018"  # '
    aspas_simples_fechamento: str = "\u2019"  # '
    
    # Travessões e hífens
    travessao: str = "\u2014"  # —
    hifen: str = "-"
    hifen_en: str = "\u2013"  # –
    
    # Espaçamentos
    espaco_pos_travessao: bool = True
    espaco_pos_pontuacao: bool = True
    
    # Regras específicas
    maiuscula_pos_travessao: bool = True
    maiuscula_pos_ponto_final: bool = True
    espaco_duplo_apos_ponto: bool = False
    
    # Lista personalizada de termos
    termos_especificos: Dict[str, str] = field(default_factory=dict)
    abreviacoes: Dict[str, str] = field(default_factory=dict)
    estrangeirismos: Dict[str, str] = field(default_factory=dict)

@dataclass
class MudancaEditorial:
    """Representa uma mudança realizada no texto."""
    tipo: str
    descricao: str
    linha: int
    coluna: int
    texto_original: str
    texto_novo: str
    regra_aplicada: str
    confianca: float  # 0.0 a 1.0
    sugestao_humano: bool = False  # Se requer revisão humana

@dataclass
class RelatorioAuditoria:
    """Relatório completo da auditoria editorial."""
    total_mudancas: int
    mudancas_por_tipo: Dict[str, int]
    padroes_identificados: List[str]
    texto_original: str
    texto_corrigido: str
    mudancas: List[MudancaEditorial]
    estatisticas: Dict[str, Any]
    recomendacoes: List[str]
```

## Fluxo de Execução

### Passo 1: Ingestão e Normalização de Texto
```python
import unicodedata

class NormalizadorTexto:
    """Motor de normalização e preparação de texto."""
    
    def __init__(self, config: ConfiguracaoEditorial):
        self.config = config
        self.mapeamento_unicode = self._criar_mapeamento_unicode()
    
    def _criar_mapeamento_unicode(self) -> Dict[str, str]:
        """Cria mapeamento de caracteres problemáticos."""
        return {
            # Aspas retas para tipográficas
            '"': self.config.aspas_abertura,
            '"': self.config.aspas_fechamento,
            '"': self.config.aspas_abertura,
            '"': self.config.aspas_fechamento,
            "'": self.config.aspas_simples_abertura,
            "'": self.config.aspas_simples_fechamento,
            
            # Travessões e hífens
            '--': self.config.travessao,
            '---': self.config.travessao,
            ' - ': f" {self.config.travessao} ",
            
            # Espaçamentos problemáticos
            '  ': ' ',  # Espaços duplos
            '\t': ' ',  # Tabulações
            '\r\n': '\n',  # Quebras de linha Windows
            '\r': '\n',  # Quebras de linha Mac antigo
        }
    
    def normalizar(self, texto: str) -> str:
        """Normaliza texto para processamento."""
        # Normalização Unicode
        texto = unicodedata.normalize('NFC', texto)
        
        # Corrigir caracteres problemáticos
        for antigo, novo in self.mapeamento_unicode.items():
            texto = texto.replace(antigo, novo)
        
        # Remover espaços em início e fim de linhas
        linhas = texto.split('\n')
        linhas = [linha.strip() for linha in linhas]
        texto = '\n'.join(linhas)
        
        # Remover linhas vazias múltiplas
        texto = re.sub(r'\n{3,}', '\n\n', texto)
        
        return texto
    
    def extrair_linhas(self, texto: str) -> List[Tuple[int, str]]:
        """Extrai linhas com numeração."""
        return [(i, linha) for i, linha in enumerate(texto.split('\n'), 1)]
```

### Passo 2: Motor de Detecção de Padrões Tipográficos
```python
class DetectorPadroesTipograficos:
    """Motor de detecção de padrões tipográficos."""
    
    def __init__(self, config: ConfiguracaoEditorial):
        self.config = config
        self.padroes = self._inicializar_padroes()
    
    def _inicializar_padroes(self) -> Dict[str, re.Pattern]:
        """Inicializa padrões regex para detecção."""
        return {
            # Aspas retas
            'aspas_retas': re.compile(r'["\']'),
            
            # Travessões incorretos
            'travessao_incorreto': re.compile(r'[-–—]{1,3}'),
            
            # Espaços duplos
            'espaco_duplo': re.compile(r'  +'),
            
            # Espaço após pontuação
            'espaco_apos_pontuacao': re.compile(r'[.!?,;:] +'),
            
            # Pontuação dobrada
            'pontuacao_dobrada': re.compile(r'[.!?,;:]{2,}'),
            
            # Espaço antes de pontuação
            'espaco_antes_pontuacao': re.compile(r' [.!?,;:]'),
            
            # Inicial maiúscula após travessão
            'inicial_apos_travessao': re.compile(f'{self.config.travessao}[a-záàãâéêíóôõúüç]'),
            
            # Diálogo sem finalização
            'dialogo_sem_finalizacao': re.compile(r'["\'].*[^.!?,;:"\']$'),
            
            # Hífen em vez de travessão em diálogos
            'hifen_dialogo': re.compile(r'["\'].*-[^-].*["\']'),
            
            # Abreviações sem ponto
            'abreviacao_sem_ponto': re.compile(r'\b(Dr|Dra|Sr|Sra|Prof|Profa|Ltd|Inc|etc)\b(?!\.)'),
            
            # Nomes próprios minúsculos após detecção de contexto
            'nome_proprio_minusculo': re.compile(r'\b(de|da|do|das|dos|e|em|com)\s+[a-záàãâéêíóôõúüç]{3,}'),
        }
    
    def detectar_problemas(self, texto: str) -> List[Dict[str, Any]]:
        """Detecta problemas tipográficos no texto."""
        problemas = []
        
        for nome_padrao, padrao in self.padroes.items():
            for match in padrao.finditer(texto):
                problema = {
                    'tipo': nome_padrao,
                    'inicio': match.start(),
                    'fim': match.end(),
                    'texto': match.group(),
                    'contexto': self._extrair_contexto(texto, match.start(), match.end())
                }
                problemas.append(problema)
        
        return sorted(problemas, key=lambda x: x['inicio'])
    
    def _extrair_contexto(self, texto: str, inicio: int, fim: int, margem: int = 20) -> str:
        """Extrai contexto ao redor do problema."""
        ctx_inicio = max(0, inicio - margem)
        ctx_fim = min(len(texto), fim + margem)
        
        contexto = texto[ctx_inicio:ctx_fim]
        if ctx_inicio > 0:
            contexto = "..." + contexto
        if ctx_fim < len(texto):
            contexto = contexto + "..."
        
        return contexto
```

### Passo 3: Motor de Padronização de Pontuação
```python
class PadronizadorPontuacao:
    """Motor de padronização de sinais de pontuação."""
    
    def __init__(self, config: ConfiguracaoEditorial):
        self.config = config
    
    def converter_aspas_retas_tipograficas(self, texto: str) -> Tuple[str, List[MudancaEditorial]]:
        """Converte aspas retas em aspas tipográficas."""
        mudancas = []
        resultado = []
        
        i = 0
        aspas_abertas = False
        
        while i < len(texto):
            char = texto[i]
            
            if char == '"':
                if not aspas_abertas:
                    resultado.append(self.config.aspas_abertura)
                    aspas_abertas = True
                    mudancas.append(MudancaEditorial(
                        tipo='conversao_aspas',
                        descricao='Aspas reta convertida para aspas de abertura tipográfica',
                        linha=0,  # Será preenchido depois
                        coluna=i,
                        texto_original='"',
                        texto_novo=self.config.aspas_abertura,
                        regra_aplicada='Padronização de aspas tipográficas',
                        confianca=0.95
                    ))
                else:
                    resultado.append(self.config.aspas_fechamento)
                    aspas_abertas = False
                    mudancas.append(MudancaEditorial(
                        tipo='conversao_aspas',
                        descricao='Aspas reta convertida para aspas de fechamento tipográfica',
                        linha=0,
                        coluna=i,
                        texto_original='"',
                        texto_novo=self.config.aspas_fechamento,
                        regra_aplicada='Padronização de aspas tipográficas',
                        confianca=0.95
                    ))
            elif char == "'":
                if not aspas_abertas:
                    resultado.append(self.config.aspas_simples_abertura)
                    aspas_abertas = True
                    mudancas.append(MudancaEditorial(
                        tipo='conversao_aspas_simples',
                        descricao='Aspas simples reta convertida para abertura tipográfica',
                        linha=0,
                        coluna=i,
                        texto_original="'",
                        texto_novo=self.config.aspas_simples_abertura,
                        regra_aplicada='Padronização de aspas tipográficas',
                        confianca=0.90
                    ))
                else:
                    resultado.append(self.config.aspas_simples_fechamento)
                    aspas_abertas = False
                    mudancas.append(MudancaEditorial(
                        tipo='conversao_aspas_simples',
                        descricao='Aspas simples reta convertida para fechamento tipográfica',
                        linha=0,
                        coluna=i,
                        texto_original="'",
                        texto_novo=self.config.aspas_simples_fechamento,
                        regra_aplicada='Padronização de aspas tipográficas',
                        confianca=0.90
                    ))
            else:
                resultado.append(char)
            
            i += 1
        
        return ''.join(resultado), mudancas
    
    def padronizar_travessoes(self, texto: str) -> Tuple[str, List[MudancaEditorial]]:
        """Padroniza uso de travessões."""
        mudancas = []
        
        # Converter hífens longos em travessões
        texto = re.sub(r'--+', self.config.travessao, texto)
        texto = re.sub(r'–+', self.config.travessao, texto)
        
        # Padronizar espaçamento após travessão
        if self.config.espaco_pos_travessao:
            texto = re.sub(
                f'{self.config.travessao}([^ \n])',
                f'{self.config.travessao} \\1',
                texto
            )
        
        # Garantir maiúscula após travessão em diálogos
        if self.config.maiuscula_pos_travessao:
            padrao = f'{self.config.travessao} ([a-záàãâéêíóôõúüç])'
            texto = re.sub(padrao, lambda m: f"{self.config.travessao} {m.group(1).upper()}", texto)
        
        return texto, mudancas
    
    def eliminar_espacos_problematicos(self, texto: str) -> Tuple[str, List[MudancaEditorial]]:
        """Elimina espaços duplos e problemáticos."""
        mudancas = []
        
        # Espaços duplos
        texto_original = texto
        texto = re.sub(r'  +', ' ', texto)
        
        if texto != texto_original:
            mudancas.append(MudancaEditorial(
                tipo='eliminacao_espaco_duplo',
                descricao='Espaços duplos eliminados',
                linha=0,
                coluna=0,
                texto_original='[espaços duplos]',
                texto_novo='[espaço único]',
                regra_aplicada='Eliminação de espaços duplos',
                confianca=1.0
            ))
        
        # Espaços em início de parágrafo (manter apenas 1)
        texto = re.sub(r'^ +', '', texto, flags=re.MULTILINE)
        
        return texto, mudancas
    
    def eliminar_pontuacao_dobrada(self, texto: str) -> Tuple[str, List[MudancaEditorial]]:
        """Elimina pontuações dobradas."""
        mudancas = []
        
        padroes_dobrados = {
            '..': '.',
            ',,': ',',
            '!!': '!',
            '??': '?',
            ';,': ';',
            ',;': ';'
        }
        
        for padrao, substituicao in padroes_dobrados.items():
            texto = texto.replace(padrao, substituicao)
        
        return texto, mudancas
```

### Passo 4: Motor de Padronização de Diálogos
```python
class PadronizadorDialogos:
    """Motor de padronização de diálogos e fala."""
    
    def __init__(self, config: ConfiguracaoEditorial):
        self.config = config
    
    def padronizar_dialogos(self, texto: str) -> Tuple[str, List[MudancaEditorial]]:
        """Padroniza diálogos conforme convenção escolhida."""
        if self.config.convencao_dialogo == ConvencaoDialogo.TRAVESSOES:
            return self._padronizar_com_travessoes(texto)
        elif self.config.convencao_dialogo == ConvencaoDialogo.ASPAS:
            return self._padronizar_com_aspas(texto)
        else:
            return texto, []
    
    def _padronizar_com_travessoes(self, texto: str) -> Tuple[str, List[MudancaEditorial]]:
        """Padroniza diálogos usando travessões."""
        mudancas = []
        linhas = texto.split('\n')
        resultado = []
        
        for linha in linhas:
            linha_padronizada = self._processar_linha_dialogo_travessao(linha)
            if linha_padronizada != linha:
                mudancas.append(MudancaEditorial(
                    tipo='padronizacao_dialogo_travessao',
                    descricao='Diálogo padronizado com travessões',
                    linha=0,
                    coluna=0,
                    texto_original=linha,
                    texto_novo=linha_padronizada,
                    regra_aplicada='Padronização de diálogos com travessões',
                    confianca=0.95
                ))
            resultado.append(linha_padronizada)
        
        return '\n'.join(resultado), mudancas
    
    def _processar_linha_dialogo_travessao(self, linha: str) -> str:
        """Processa uma linha de diálogo com travessões."""
        # Detectar início de diálogo (aspas)
        padrao_aspas = re.compile(r'^(.*?)["\u201c\u201d](.*?)["\u201c\u201d](.*)$')
        match = padrao_aspas.match(linha)
        
        if match:
            antes, dialogo, depois = match.groups()
            
            # Converter para travessão
            linha_nova = f"{antes}{self.config.travessao} {dialogo}{self.config.travessao}{depois}"
            
            # Garantir maiúscula após travessão
            padrao_maiuscula = re.compile(f'{self.config.travessao} ([a-záàãâéêíóôõúüç])')
            linha_nova = padrao_maiuscula.sub(
                lambda m: f"{self.config.travessao} {m.group(1).upper()}",
                linha_nova
            )
            
            return linha_nova
        
        return linha
    
    def _padronizar_com_aspas(self, texto: str) -> Tuple[str, List[MudancaEditorial]]:
        """Padroniza diálogos usando aspas."""
        mudancas = []
        linhas = texto.split('\n')
        resultado = []
        
        for linha in linhas:
            linha_padronizada = self._processar_linha_dialogo_aspas(linha)
            if linha_padronizada != linha:
                mudancas.append(MudancaEditorial(
                    tipo='padronizacao_dialogo_aspas',
                    descricao='Diálogo padronizado com aspas',
                    linha=0,
                    coluna=0,
                    texto_original=linha,
                    texto_novo=linha_padronizada,
                    regra_aplicada='Padronização de diálogos com aspas',
                    confianca=0.95
                ))
            resultado.append(linha_padronizada)
        
        return '\n'.join(resultado), mudancas
    
    def _processar_linha_dialogo_aspas(self, linha: str) -> str:
        """Processa uma linha de diálogo com aspas."""
        # Garantir aspas tipográficas corretas
        # Aspas de abertura devem ser curvas para cima
        # Aspas de fechamento devem ser curvas para baixo
        
        resultado = []
        aspas_abertas = False
        
        for char in linha:
            if char == '"':
                if not aspas_abertas:
                    resultado.append(self.config.aspas_abertura)
                    aspas_abertas = True
                else:
                    resultado.append(self.config.aspas_fechamento)
                    aspas_abertas = False
            else:
                resultado.append(char)
        
        return ''.join(resultado)
    
    def validar_pontuacao_dialogos(self, texto: str) -> List[MudancaEditorial]:
        """Valida pontuação em diálogos."""
        mudancas = []
        
        # Verificar se há diálogos sem finalização
        padrao_dialogo_aberto = re.compile(r'["\u201c].*[^.!?,;:"\u201d]$')
        
        for i, linha in enumerate(texto.split('\n'), 1):
            if padrao_dialogo_aberto.match(linha):
                mudancas.append(MudancaEditorial(
                    tipo='dialogo_sem_finalizacao',
                    descricao='Diálio pode estar sem finalização adequada',
                    linha=i,
                    coluna=0,
                    texto_original=linha,
                    texto_novo='',
                    regra_aplicada='Validação de pontuação em diálogos',
                    confianca=0.70,
                    sugestao_humano=True
                ))
        
        return mudancas
```

### Passo 5: Motor de Padronização Editorial
```python
class PadronizadorEditorial:
    """Motor de padronização de elementos editoriais."""
    
    def __init__(self, config: ConfiguracaoEditorial):
        self.config = config
    
    def padronizar_maiusculas(self, texto: str) -> Tuple[str, List[MudancaEditorial]]:
        """Padroniza uso de maiúsculas em nomes próprios e termos específicos."""
        mudancas = []
        
        # Termos específicos que devem ter maiúscula inicial
        termos_maiusculas = {
            'governo': 'Governo',
            'estado': 'Estado',
            'universidade': 'Universidade',
            'presidente': 'Presidente',
            'ministro': 'Ministro',
            'lei': 'Lei',
            'constituição': 'Constituição',
            'sociedade': 'Sociedade',
            'cultura': 'Cultura',
            'economia': 'Economia',
            'política': 'Política'
        }
        
        for termo, correcao in termos_maiusculas.items():
            padrao = re.compile(rf'\b{termo}\b', re.IGNORECASE)
            
            for match in padrao.finditer(texto):
                # Verificar se está no início de frase ou é nome próprio
                if self._deve_ser_maiuscula(texto, match.start()):
                    texto_corrigido = texto[:match.start()] + correcao + texto[match.end():]
                    if texto_corrigido != texto:
                        mudancas.append(MudancaEditorial(
                            tipo='correcao_maiuscula',
                            descricao=f'Termo "{termo}" padronizado para "{correcao}"',
                            linha=0,
                            coluna=match.start(),
                            texto_original=match.group(),
                            texto_novo=correcao,
                            regra_aplicada='Padronização de maiúsculas em termos específicos',
                            confianca=0.85
                        ))
                        texto = texto_corrigido
        
        return texto, mudancas
    
    def _deve_ser_maiuscula(self, texto: str, posicao: int) -> bool:
        """Verifica se uma palavra deve ter maiúscula."""
        # Verificar se está no início de frase
        if posicao == 0:
            return True
        
        # Verificar caractere anterior
        char_anterior = texto[posicao - 1]
        if char_anterior in '.!?\n':
            return True
        
        # Verificar se é nome próprio (após preposição ou artigo)
        contexto = texto[max(0, posicao - 20):posicao]
        padroes_nome_proprio = [
            r'(?:de|da|do|das|dos|em|com|para|por)\s+$',
            r'(?:o|a|os|as|um|uma|uns|umas)\s+$'
        ]
        
        for padrao in padroes_nome_proprio:
            if re.search(padrao, contexto, re.IGNORECASE):
                return True
        
        return False
    
    def padronizar_abreviacoes(self, texto: str) -> Tuple[str, List[MudancaEditorial]]:
        """Padroniza abreviações."""
        mudancas = []
        
        abreviacoes_padrao = {
            'Dr': 'Dr.',
            'Dra': 'Dra.',
            'Sr': 'Sr.',
            'Sra': 'Sra.',
            'Prof': 'Prof.',
            'Profa': 'Profa.',
            'Ltd': 'Ltd.',
            'Inc': 'Inc.',
            'etc': 'etc.'
        }
        
        # Adicionar abreviações personalizadas
        abreviacoes_padrao.update(self.config.abreviacoes)
        
        for abrev, correcao in abreviacoes_padrao.items():
            padrao = re.compile(rf'\b{abrev}\b(?!\.)', re.IGNORECASE)
            
            for match in padrao.finditer(texto):
                texto_corrigido = texto[:match.start()] + correcao + texto[match.end():]
                if texto_corrigido != texto:
                    mudancas.append(MudancaEditorial(
                        tipo='correcao_abreviacao',
                        descricao=f'Abreviação "{abrev}" padronizada para "{correcao}"',
                        linha=0,
                        coluna=match.start(),
                        texto_original=match.group(),
                        texto_novo=correcao,
                        regra_aplicada='Padronização de abreviações',
                        confianca=0.90
                    ))
                    texto = texto_corrigido
        
        return texto, mudancas
    
    def padronizar_estrangeirismos(self, texto: str) -> Tuple[str, List[MudancaEditorial]]:
        """Normaliza estrangeirismos conforme guia de estilo."""
        mudancas = []
        
        # Estrangeirismos comuns com normas
        estrangeirismos_padrao = {
            'software': 'software',
            'hardware': 'hardware',
            'website': 'sítio eletrônico',
            'e-mail': 'mensagem eletrônica',
            'on-line': 'em linha',
            'download': 'transferência',
            'upload': 'envio',
            'blog': 'sítio eletrônico',
            'podcast': 'transmissão',
            'selfie': 'automensagem',
            'marketing': 'comercialização',
            'management': 'administração',
            'business': 'negócio',
            'cash': 'dinheiro',
            'staff': 'equipe',
            'briefing': 'reunião preparatória'
        }
        
        # Adicionar estrangeirismos personalizados
        estrangeirismos_padrao.update(self.config.estrangeirismos)
        
        for termo, correcao in estrangeirismos_padrao.items():
            if termo != correcao:  # Apenas se houver tradução
                padrao = re.compile(rf'\b{re.escape(termo)}\b', re.IGNORECASE)
                
                for match in padrao.finditer(texto):
                    # Verificar se está entre aspas (pode ser intencional)
                    if not self._esta_entre_aspas(texto, match.start()):
                        texto_corrigido = texto[:match.start()] + correcao + texto[match.end():]
                        if texto_corrigido != texto:
                            mudancas.append(MudancaEditorial(
                                tipo='correcao_estrangeirismo',
                                descricao=f'Estrangeirismo "{termo}" normalizado para "{correcao}"',
                                linha=0,
                                coluna=match.start(),
                                texto_original=match.group(),
                                texto_novo=correcao,
                                regra_aplicada='Normalização de estrangeirismos',
                                confianca=0.80,
                                sugestao_humano=True
                            ))
                            texto = texto_corrigido
        
        return texto, mudancas
    
    def _esta_entre_aspas(self, texto: str, posicao: int) -> bool:
        """Verifica se uma posição está entre aspas."""
        antes = texto[:posicao]
        
        # Contar aspas antes da posição
        aspas_abertas = antes.count(self.config.aspas_abertura)
        aspas_fechadas = antes.count(self.config.aspas_fechamento)
        
        return aspas_abertas > aspas_fechadas
```

### Passo 6: Motor de Análise Rítmica
```python
class AnalisadorRitmico:
    """Motor de análise rítmica e de fluidez do texto."""
    
    def __init__(self, config: ConfiguracaoEditorial):
        self.config = config
    
    def analisar_fluidez(self, texto: str) -> List[MudancaEditorial]:
        """Analisa fluidez e ritmo do texto."""
        mudancas = []
        
        # Detectar sobrecarga de pontuação
        mudancas.extend(self._detectar_sobrecarga_pontuacao(texto))
        
        # Detectar frases muito longas
        mudancas.extend(self._detectar_frases_longas(texto))
        
        # Detectar repetições excessivas
        mudancas.extend(self._detectar_repeticoes(texto))
        
        return mudancas
    
    def _detectar_sobrecarga_pontuacao(self, texto: str) -> List[MudancaEditorial]:
        """Detecta sobrecarga de pausas gramaticais."""
        mudancas = []
        
        # Padrão: múltiplas vírgulas em sequência
        padrao = re.compile(r'([^.!?:;,])\s*,\s*,\s*,')
        
        for match in padrao.finditer(texto):
            mudancas.append(MudancaEditorial(
                tipo='sobrecarga_pontuacao',
                descricao='Possível sobrecarga de vírgulas prejudicando fluidez',
                linha=0,
                coluna=match.start(),
                texto_original=match.group(),
                texto_novo='',
                regra_aplicada='Análise rítmica - fluidez',
                confianca=0.75,
                sugestao_humano=True
            ))
        
        return mudancas
    
    def _detectar_frases_longas(self, texto: str, limite_palavras: int = 50) -> List[MudancaEditorial]:
        """Detecta frases excessivamente longas."""
        mudancas = []
        
        # Dividir em frases
        frases = re.split(r'[.!?]+', texto)
        
        for i, frase in enumerate(frases, 1):
            palavras = frase.split()
            if len(palavras) > limite_palavras:
                mudancas.append(MudancaEditorial(
                    tipo='frase_longa',
                    descricao=f'Frase com {len(palavras)} palavras pode estar excessivamente longa',
                    linha=0,
                    coluna=0,
                    texto_original=frase.strip(),
                    texto_novo='',
                    regra_aplicada='Análise rítmica - comprimento de frases',
                    confianca=0.70,
                    sugestao_humano=True
                ))
        
        return mudancas
    
    def _detectar_repeticoes(self, texto: str, limite: int = 3) -> List[MudancaEditorial]:
        """Detecta repetições excessivas de palavras ou expressões."""
        mudancas = []
        
        # Dividir em palavras
        palavras = re.findall(r'\b\w+\b', texto.lower())
        
        # Contar repetições consecutivas
        contagem = {}
        palavra_atual = None
        count = 0
        
        for palavra in palavras:
            if palavra == palavra_atual:
                count += 1
            else:
                if count >= limite and palavra_atual:
                    mudancas.append(MudancaEditorial(
                        tipo='repeticao_excessiva',
                        descricao=f'Palavra "{palavra_atual}" repetida {count} vezes consecutivas',
                        linha=0,
                        coluna=0,
                        texto_original=f'[{palavra_atual}] x{count}',
                        texto_novo='',
                        regra_aplicada='Análise rítmica - repetições',
                        confianca=0.80,
                        sugestao_humano=True
                    ))
                palavra_atual = palavra
                count = 1
        
        return mudancas
```

### Passo 7: Motor de Relatórios de Auditoria
```python
class GeradorRelatorios:
    """Motor de geração de relatórios de auditoria."""
    
    def __init__(self):
        pass
    
    def gerar_relatorio(self, mudancas: List[MudancaEditorial], 
                       texto_original: str, texto_corrigido: str) -> RelatorioAuditoria:
        """Gera relatório completo de auditoria."""
        
        # Agrupar mudanças por tipo
        mudancas_por_tipo = {}
        for m in mudancas:
            if m.tipo not in mudancas_por_tipo:
                mudancas_por_tipo[m.tipo] = 0
            mudancas_por_tipo[m.tipo] += 1
        
        # Identificar padrões
        padroes = self._identificar_padroes(mudancas)
        
        # Calcular estatísticas
        estatisticas = self._calcular_estatisticas(texto_original, texto_corrigido, mudancas)
        
        # Gerar recomendações
        recomendacoes = self._gerar_recomendacoes(mudancas, estatisticas)
        
        return RelatorioAuditoria(
            total_mudancas=len(mudancas),
            mudancas_por_tipo=mudancas_por_tipo,
            padroes_identificados=padroes,
            texto_original=texto_original,
            texto_corrigido=texto_corrigido,
            mudancas=mudancas,
            estatisticas=estatisticas,
            recomendacoes=recomendacoes
        )
    
    def _identificar_padroes(self, mudancas: List[MudancaEditorial]) -> List[str]:
        """Identifica padrões recorrentes nas mudanças."""
        padroes = []
        
        # Contar tipos de mudança
        contagem = {}
        for m in mudancas:
            contagem[m.tipo] = contagem.get(m.tipo, 0) + 1
        
        # Identificar padrões significativos
        if contagem.get('conversao_aspas', 0) > 5:
            padroes.append('Uso frequente de aspas retas - possível padrão de digitação')
        
        if contagem.get('espaco_duplo', 0) > 3:
            padroes.append('Espaços duplos frequentes - possível problema de formatação')
        
        if contagem.get('correcao_maiuscula', 0) > 10:
            padroes.append('Muitas correções de maiúsculas - possível falta de padronização inicial')
        
        if contagem.get('dialogo_sem_finalizacao', 0) > 2:
            padroes.append('Diálogos sem finalização - possível estilo narrativo ou erro')
        
        return padroes
    
    def _calcular_estatisticas(self, texto_original: str, texto_corrigido: str, 
                              mudancas: List[MudancaEditorial]) -> Dict[str, Any]:
        """Calcula estatísticas da auditoria."""
        palavras_originais = len(texto_original.split())
        palavras_corrigidas = len(texto_corrigido.split())
        
        return {
            'total_caracteres_original': len(texto_original),
            'total_caracteres_corrigido': len(texto_corrigido),
            'total_palavras_original': palavras_originais,
            'total_palavras_corrigido': palavras_corrigidas,
            'total_mudancas': len(mudancas),
            'mudancas_por_tipo': {},
            'confianca_media': sum(m.confianca for m in mudancas) / len(mudancas) if mudancas else 0,
            'mudancas_requerem_revisao_humana': sum(1 for m in mudancas if m.sugestao_humano)
        }
    
    def _gerar_recomendacoes(self, mudancas: List[MudancaEditorial], 
                           estatisticas: Dict[str, Any]) -> List[str]:
        """Gera recomendações baseadas na auditoria."""
        recomendacoes = []
        
        if estatisticas['confianca_media'] < 0.8:
            recomendacoes.append('Confiança média das correções está baixa - revisar manualmente')
        
        if estatisticas['mudancas_requerem_revisao_humana'] > 10:
            recomendacoes.append('Muitas correções requerem revisão humana - considere ajustar configurações')
        
        # Recomendações específicas por tipo
        tipos_mudanca = [m.tipo for m in mudancas]
        
        if 'conversao_aspas' in tipos_mudanca:
            recomendacoes.append('Considere configurar o editor para inserir aspas tipográficas automaticamente')
        
        if 'espaco_duplo' in tipos_mudanca:
            recomendacoes.append('Configure o editor para eliminar espaços duplos automaticamente')
        
        if 'correcao_estrangeirismo' in tipos_mudanca:
            recomendacoes.append('Revise os estrangeirismos substituídos - alguns podem ser intencionais')
        
        return recomendacoes
```

### Passo 8: Pipeline Completo de Copydesk
```python
class PipelineCopydesk:
    """Pipeline completo de auditoria e refinamento editorial."""
    
    def __init__(self, config: ConfiguracaoEditorial = None):
        self.config = config or ConfiguracaoEditorial()
        self.normalizador = NormalizadorTexto(self.config)
        self.detector = DetectorPadroesTipograficos(self.config)
        self.padronizador_pontuacao = PadronizadorPontuacao(self.config)
        self.padronizador_dialogos = PadronizadorDialogos(self.config)
        self.padronizador_editorial = PadronizadorEditorial(self.config)
        self.analisador_ritmico = AnalisadorRitmico(self.config)
        self.gerador_relatorios = GeradorRelatorios()
    
    def executar_auditoria(self, texto: str, 
                          nome_arquivo: str = "documento") -> Tuple[str, RelatorioAuditoria]:
        """
        Executa auditoria completa no texto.
        
        Args:
            texto: Texto a ser auditado
            nome_arquivo: Nome do arquivo para referência
        
        Returns:
            Tuple com texto corrigido e relatório
        """
        # 1. Normalizar texto
        texto_normalizado = self.normalizador.normalizar(texto)
        
        # 2. Converter aspas retas em tipográficas
        texto_aspas, mudancas_aspas = self.padronizador_pontuacao.converter_aspas_retas_tipograficas(texto_normalizado)
        
        # 3. Padronizar travessões
        texto_travessoes, mudancas_travessoes = self.padronizador_pontuacao.padronizar_travessoes(texto_aspas)
        
        # 4. Eliminar espaços problemáticos
        texto_espacos, mudancas_espacos = self.padronizador_pontuacao.eliminar_espacos_problematicos(texto_travessoes)
        
        # 5. Eliminar pontuação dobrada
        texto_pontuacao, mudancas_pontuacao = self.padronizador_pontuacao.eliminar_pontuacao_dobrada(texto_espacos)
        
        # 6. Padronizar diálogos
        texto_dialogos, mudancas_dialogos = self.padronizador_dialogos.padronizar_dialogos(texto_pontuacao)
        
        # 7. Validar pontuação em diálogos
        mudancas_validacao = self.padronizador_dialogos.validar_pontuacao_dialogos(texto_dialogos)
        
        # 8. Padronizar maiúsculas
        texto_maiusculas, mudancas_maiusculas = self.padronizador_editorial.padronizar_maiusculas(texto_dialogos)
        
        # 9. Padronizar abreviações
        texto_abreviacoes, mudancas_abreviacoes = self.padronizador_editorial.padronizar_abreviacoes(texto_maiusculas)
        
        # 10. Padronizar estrangeirismos
        texto_estrangeirismos, mudancas_estrangeirismos = self.padronizador_editorial.padronizar_estrangeirismos(texto_abreviacoes)
        
        # 11. Análise rítmica
        mudancas_ritmo = self.analisador_ritmico.analisar_fluidez(texto_estrangeirismos)
        
        # 12. Combinar todas as mudanças
        todas_mudancas = (
            mudancas_aspas + 
            mudancas_travessoes + 
            mudancas_espacos + 
            mudancas_pontuacao + 
            mudancas_dialogos + 
            mudancas_validacao + 
            mudancas_maiusculas + 
            mudancas_abreviacoes + 
            mudancas_estrangeirismos + 
            mudancas_ritmo
        )
        
        # 13. Gerar relatório
        relatorio = self.gerador_relatorios.gerar_relatorio(
            todas_mudancas,
            texto,
            texto_estrangeirismos
        )
        
        return texto_estrangeirismos, relatorio
    
    def processar_arquivo(self, caminho_entrada: Path, caminho_saida: Path = None) -> Tuple[Path, RelatorioAuditoria]:
        """
        Processa arquivo completo.
        
        Args:
            caminho_entrada: Caminho do arquivo de entrada
            caminho_saida: Caminho do arquivo de saída (opcional)
        
        Returns:
            Tuple com caminho do arquivo processado e relatório
        """
        # Ler arquivo
        with open(caminho_entrada, 'r', encoding='utf-8') as f:
            texto = f.read()
        
        # Executar auditoria
        texto_corrigido, relatorio = self.executar_auditoria(texto, caminho_entrada.name)
        
        # Definir caminho de saída
        if caminho_saida is None:
            caminho_saida = caminho_entrada.parent / f"{caminho_entrada.stem}_corrigido{caminho_entrada.suffix}"
        
        # Salvar texto corrigido
        with open(caminho_saida, 'w', encoding='utf-8') as f:
            f.write(texto_corrigido)
        
        # Salvar relatório
        caminho_relatorio = caminho_saida.parent / f"{caminho_saida.stem}_relatorio.md"
        self._salvar_relatorio(relatorio, caminho_relatorio)
        
        return caminho_saida, relatorio
    
    def _salvar_relatorio(self, relatorio: RelatorioAuditoria, caminho: Path):
        """Salva relatório em formato Markdown."""
        with open(caminho, 'w', encoding='utf-8') as f:
            f.write("# Relatório de Auditoria Editorial\n\n")
            f.write(f"**Total de mudanças:** {relatorio.total_mudancas}\n\n")
            
            f.write("## Mudanças por Tipo\n\n")
            for tipo, contagem in relatorio.mudancas_por_tipo.items():
                f.write(f"- **{tipo}:** {contagem}\n")
            
            f.write("\n## Padrões Identificados\n\n")
            for padrao in relatorio.padroes_identificados:
                f.write(f"- {padrao}\n")
            
            f.write("\n## Estatísticas\n\n")
            for chave, valor in relatorio.estatisticas.items():
                if isinstance(valor, float):
                    f.write(f"- **{chave}:** {valor:.2f}\n")
                else:
                    f.write(f"- **{chave}:** {valor}\n")
            
            f.write("\n## Recomendações\n\n")
            for recomendacao in relatorio.recomendacoes:
                f.write(f"- {recomendacao}\n")
            
            f.write("\n## Detalhes das Mudanças\n\n")
            for i, m in enumerate(relatorio.mudancas[:50], 1):  # Limitar a 50
                f.write(f"### Mudança {i}\n")
                f.write(f"- **Tipo:** {m.tipo}\n")
                f.write(f"- **Descrição:** {m.descricao}\n")
                f.write(f"- **Confiança:** {m.confianca:.2f}\n")
                if m.sugestao_humano:
                    f.write(f"- **⚠️ Requer revisão humana**\n")
                f.write(f"- **Texto original:** `{m.texto_original}`\n")
                if m.texto_novo:
                    f.write(f"- **Texto novo:** `{m.texto_novo}`\n")
                f.write("\n")
            
            if len(relatorio.mudancas) > 50:
                f.write(f"\n*... e mais {len(relatorio.mudancas) - 50} mudanças no arquivo completo.*\n")
```

## Tratamento de Erros
- **Arquivo não encontrado**: Verificar caminho e permissões
- **Encoding inválido**: Tentar UTF-8, Latin-1, CP1252 automaticamente
- **Texto vazio**: Informar e solicitar conteúdo
- **Configuração inválida**: Usar configuração padrão
- **Muitas mudanças**: Dividir processamento em blocos menores

## Regras
### O que SEMPRE fazer
- Manter consistência na convenção de diálogos escolhida
- Converter aspas retas em tipográficas sempre
- Garantir maiúscula após travessão em diálogos
- Eliminar espaços duplos e pontuações dobradas
- Gerar relatório detalhado de todas as intervenções
- Marcar correções de baixa confiança para revisão humana

### O que NUNCA fazer
- Alterar conteúdo ou significado do texto
- Modificar estrutura de frases ou parágrafos
- Alterar nomes próprios sem certeza absoluta
- Remover pontuação sem contexto adequado
- Aplicar correções automáticas sem registro no relatório

### Quando algo falhar
1. **Detecção ambígua**: Marcar para revisão humana com contexto
2. **Muitas exceções**: Aumentar nível de confiança mínimo
3. **Padrão não reconhecido**: Ignorar e registrar no relatório
4. **Conflito de regras**: Priorizar regra mais específica

## Exemplos de Uso

### Exemplo 1: Auditoria completa
```python
config = ConfiguracaoEditorial(
    convencao_dialogo=ConvencaoDialogo.TRAVESSOES,
    convencao_pontuacao=ConvencaoPontuacao.PORTUGUES_BRASIL
)
pipeline = PipelineCopydesk(config)
texto_corrigido, relatorio = pipeline.executar_auditoria(texto_original)
print(f"Total de mudanças: {relatorio.total_mudancas}")
```

### Exemplo 2: Processar arquivo
```python
pipeline = PipelineCopydesk()
arquivo_corrigido, relatorio = pipeline.processar_arquivo(
    Path('manuscrito.txt'),
    Path('manuscrito_corrigido.txt')
)
```

### Exemplo 3: Configuração personalizada
```python
config = ConfiguracaoEditorial(
    convencao_dialogo=ConvencaoDialogo.ASPAS,
    termos_especificos={'meio ambiente': 'Meio Ambiente'},
    abreviacoes={'Prof.': 'Prof.º'}
)
pipeline = PipelineCopydesk(config)
```
