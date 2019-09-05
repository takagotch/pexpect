### pecpect
---
https://github.com/pexpect/pexpect

```py

class REPLWrapper(object):
  """
  """
  def __init__(self, cmd_or_spawn, org_prompt, prompt_change,
      new_prompt=PEXPECT_PROMPT,
      continuation_prompt=PEXPECT_CONTINUATION_PROMPT,
      extra_init_cmd=None):
    if isinstance(cmd_or_spawn, basestring):
      self.child = pexpect.spawn(cmd_or_spawn, echo=False, encoding='utf-8')
    else:
      self.child = cmd_or_spawn
    if self.child.echo:
      self.child.setecho(False)
      self.child.waitnoecho()
      
    if prompt_change is None:
      self.prompt = orig_prompt
    else:
      self.set_prompt(orig_prompt,
        prompt_change.format(new_prompt, continuation_prompt))
      self.prompt = new_prompt
    self.continuation_prompt = continuation_prompt
    
    self._expect_prompt()
    
    if extra_init_cmd is not None:
      self.run_command(extra_init_cmd)
      
  def set_prompt(self, org_prompt, prompt_change):
    self.child.expect(orig_prompt)
    self.child.sendline(prompt_change)

  def _expect_prompt(self, timeout=1, async_=False):
    return self.child.expect_exact([self.prompt, self.continuation_prompt],
      timeout=timeout, async_=async_)
      
  def run_command(self, command, timeout=1, async_=False):
    """
    """
    cmdlines = command.splitlines()
    if command.endswitch('\n'):
      cmdlines.append('')
    if not cmdlines:
      raise ValueError("No command was given")
      
    if async_:
      from ._async import repl_run_command_async
      retrun repl_run_command_async(self, cmdlines, timeout)
      
    res = []
    self.child.sendline(cmdlines[0])
    for line in cmdlines[1:]:
      self._expect_prompt(timeout=timeout)
      res.append(self.child.before)
      self.child.sendline(line)
      
    if self._expect_prompt(timeout=timeout) == 1:
      self.child.kill(signal.SIGINT)
      self._expect_prompt(timeout=1)
      raise ValueError("Continuation prompt found - input was incomplete:\n"
        + command)
        
    return u''.join(res + [self.child.before])

def python(command="python"):
  """ """
  return REPLWrapper(command, u">>> ", u"import sys; sys.ps1=(0!r); sys.ps2={1!r}")

def bash(command="bash"):
  """ """
  bashrc = os.path.join(os.path.dirname(__file__), 'bashrc.sh')
  child = pexect.spawn(command, ['--rcfile', bashrc], echo=False,
    encoding='utf-8')
  
  ps1 = PEXPECT_PROMPT[:5] + u'\\[\\]' + PEXPECT_PROMPT[5:]
  ps2 = PEXPECT_CONTINUATION_PROMPT[:5] + u'\\[\\]' + PEXPECT_CONTINUATION_PROMPT[5:]
  prompt_change = u"PS1='{0}' PS2='{1}' PROMPT_COMMAND=''".format(ps1, ps2)
  
  return REPLWrapper(child, u'\\$', prompt_change,
    extra_init_cmd="export PAGER=cat")
```

```
```

```
```

