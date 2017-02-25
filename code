import collections
import re
import operator as op
import numpy as np
Token = collections.namedtuple('Token', ['typ', 'value', 'line', 'column'])

def tokenize(s):
    keywords = {'if', 'case', 'end'}
    Token_regex = [
        ('Number',  r'\d+(\.\d+)?'), 
        ('ASSIGN',  r'='),('Block_Change',  r';'),          
        ('delimiter',     r'[,.]'),('Block_Start',     r'->'),           
        ('variable',      r'[A-Z]+[A-Za-z]*'),('atom',      r'[A-Za-z]+'),
        ('op_code',      r'[+*\/-]'),('log_code',      r'[\<\>]'),    
        ('SKIP', r'[ \t]'),
        ('NEWLINE',r'\n')        
    ]
    tok_regex = '|'.join('(?P<%s>%s)' % pair for pair in Token_regex)
    line = 1
    pos = line_start = 0
    get_token = re.compile(tok_regex).match
    key = get_token(s)
    print(key.group())
    while key is not None:
        typ = key.lastgroup
        if typ != 'SKIP':
            val = key.group(typ)
            if typ == 'atom' and val in keywords:
                typ = val
            if(typ!='\n'):
                yield Token(typ, val, line, key.start()-line_start)
        pos = key.end()
        key = get_token(s,pos)
def is_present(token,op):
    try:
        key=token.index(op)
        return key
    except:
        return -1
def is_ID(token):
    if(len(token)==1 and (token[0]=='Number' or token[0]=='variable')):
         return True
    else:
        return False
def expr4(token):
    parse=[]
    if(is_ID(token[0])):
        return token[1],True
    key=is_present(token[1],'*')
    if(key!=-1 and (key+1)!=len(token[0]) and key != 0):
        parse.append('*')
        medl=np.array(token)[0:,:key].tolist()
        medr=np.array(token)[0:,key+1:].tolist()
        resl,decl=expr1(medl)
        resr,decr=expr1(medr)
        if(decl and decr):
            parse.append(resl)
            parse.append(resr)
            return parse,True
        else:
            return [],False
    else:
        return [],False

def expr3(token):
    parse=[]
    if(is_ID(token[0])):
        return token[1],True
    key=is_present(token[1],'/')
    if(key!=-1 and (key+1)!=len(token[0]) and key != 0):
        parse.append('/')
        medl=np.array(token)[0:,:key].tolist()
        medr=np.array(token)[0:,key+1:].tolist()
        resl,decl=expr1(medl)
        resr,decr=expr1(medr)
        if(decl and decr):
            parse.append(resl)
            parse.append(resr)
            return parse,True
        else:
            return [],False
    else:
        return expr4(token)
def expr2(token):
    parse=[]
    if(is_ID(token[0])):
        return token[1],True
    key=is_present(token[1],'-')
    if(key!=-1 and (key+1)!=len(token[0]) and key != 0):
        parse.append('-')
        medl=np.array(token)[0:,:key].tolist()
        medr=np.array(token)[0:,key+1:].tolist()
        resl,decl=expr1(medl)
        resr,decr=expr1(medr)
        if(decl and decr):
            parse.append(resl)
            parse.append(resr)
            return parse,True
        else:
            return [],False
    else:
        return expr3(token)

def expr1(token):
    parse=[]
    if(len(token[0])==0):
        return [],False 
    if(is_ID(token[0])):
        return token[1],True
    key=is_present(token[1],'+')
    if(key!=-1 and (key+1)!=len(token[0]) and key != 0):
        parse.append('+')
        medl=np.array(token)[0:,:key].tolist()
        medr=np.array(token)[0:,key+1:].tolist()
        resl,decl=expr1(medl)
        resr,decr=expr1(medr)
        if(decl and decr):
            parse.append(resl)
            parse.append(resr)
            return parse,True
        else:
            return [],False
    else:
        return expr2(token)
        
def expr(token):
    parse=[]
    key=is_present(token[1],'=')
    if(is_ID(token[0][:key]) and key == 1):
        parse.append(token[1][key])
        parse.append(token[1][:key])
        med=np.array(token)[0:,key+1:].tolist()
        res,dec=expr1(med)
        if(dec):
            parse.append(res)
            return parse,True
        else:
            return [],False
    else:
        return [],False
def parse_if_clause(token):
    print(token)
    return token,True
def parse_if_clauses(token):
    parse=[]
    if(len(token[0])==0):
        return [],False 
    key=is_present(token[1],';')
    if(key!=-1 and (key+1)!=len(token[0]) and key != 0):
        
        medl=np.array(token)[0:,:key].tolist()
        medr=np.array(token)[0:,key+1:].tolist()
        resl,decl=parse_if_clauses(medl)
        resr,decr=parse_if_clauses(medr)
        if(decl and decr):
            parse.append(resl)
            parse.append(resr)
            return parse,True
        else:
            return [],False
    else:
        return parse_if_clause(token)        
def blocks(chain):
    prog=[]
    while(len(chain)!=0):
        if(chain[0].typ=='if'):
            token_block=[]
            while(len(chain)!=0):
                token_block.append(chain[0])
                if(chain[0].typ=='end'):
                    break
                chain.pop(0)
            token_block.pop(0)
            token_block.pop(-1)
            token_block=map(list, zip(*token_block))
            #print(token_block)
            parse,dec=parse_if_clauses(token_block[:2])
            prog.append(parse)
        else:
            token_block=[]
            while(len(chain)!=0):
                if (chain[0].value==',' or chain[0].value=='.'):
                    chain.pop(0)
                    break
                token_block.append(chain[0])
                chain.pop(0)
            token_block=map(list, zip(*token_block))
            parse,dec=expr(token_block[:2])
            prog.append(parse)
    return prog
            
program = '''
    if A>20 -> B=5; true ->B=10 end.
'''
token_list=[]
for token in tokenize(program):
    if(token.typ!='NEWLINE'):
        token_list.append(token)
print(blocks(token_list))
