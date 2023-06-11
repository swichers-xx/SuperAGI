from abc import ABC, abstractmethod
from typing import Any, Callable, Dict, Optional, Tuple, Type, Union
from pydantic import BaseModel, Field, create_model
import os
import shlex
import subprocess

class FunctionalToolArgsSchema(BaseModel):
    cmd: str = Field(...)
    env: Optional[Dict[str, str]] = Field(default_factory=dict)
    save_to_file: Optional[bool] = Field(default=False)
    filename: Optional[str] = Field(default=None)

class BaseTool(ABC, BaseModel):
    name: str = Field(default=None)
    description: str = Field(default=None)
    args_schema: Type[BaseModel] = Field(default=None)

    @abstractmethod
    def execute(self, *args: Any, **kwargs: Any) -> Any:
        pass

class FunctionalTool(BaseTool):
    func: Callable[..., Any]

    def execute(self, *args: Any, **kwargs: Any) -> Any:
        return self.func(*args, **kwargs)

def run_command(cmd: str, env: Optional[Dict[str, str]] = None, save_to_file: Optional[bool] = False, filename: Optional[str] = None) -> Tuple[str, str]:
    env_vars = os.environ.copy()
    if env is not None:
        env_vars.update(env)
    cmd_parts = shlex.split(cmd)
    result = subprocess.run(cmd_parts, stdout=subprocess.PIPE, stderr=subprocess.PIPE, env=env_vars)
    output = result.stdout.decode()
    error = result.stderr.decode()
    if save_to_file:
        filename = filename or "command.py"
        with open(filename, 'w') as f:
            f.write(f"import os\n\nos.system('{cmd}')\n")
    return output, error

run_command_tool = FunctionalTool(
    name="run_command",
    description="A tool to run shell commands",
    args_schema=FunctionalToolArgsSchema,
    func=run_command
)

if __name__ == "__main__":
    command = input("Enter the command you want to run: ")
    output, error = run_command_tool.execute({"cmd": command})
    if output:
        print("Output:\n", output)
    if error:
        print("Error:\n", error)
