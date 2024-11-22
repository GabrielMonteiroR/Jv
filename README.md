
---

### 1. **Entidade: `Subcategoria`**

```java
package com.example.demo.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Subcategoria {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    // Getters e Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getNome() {
        return nome;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }
}
```

---

### 2. **DTOs**

#### `SubcategoriaDTO` (Usado para leitura)

```java
package com.example.demo.dto;

public class SubcategoriaDTO {
    private Long id;
    private String nome;

    // Getters e Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getNome() {
        return nome;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }
}
```

#### `SubcategoriaRequestDTO` (Usado para criação/atualização)

```java
package com.example.demo.dto;

import javax.validation.constraints.NotBlank;

public class SubcategoriaRequestDTO {

    @NotBlank(message = "O nome é obrigatório.")
    private String nome;

    // Getters e Setters
    public String getNome() {
        return nome;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }
}
```

---

### 3. **Mapper com MapStruct**

```java
package com.example.demo.mapper;

import com.example.demo.dto.SubcategoriaDTO;
import com.example.demo.dto.SubcategoriaRequestDTO;
import com.example.demo.model.Subcategoria;
import org.mapstruct.Mapper;

@Mapper(componentModel = "spring")
public interface SubcategoriaMapper {
    SubcategoriaDTO toDTO(Subcategoria subcategoria);

    Subcategoria toEntity(SubcategoriaRequestDTO dto);
}
```

---

### 4. **Repositório**

```java
package com.example.demo.repository;

import com.example.demo.model.Subcategoria;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface SubcategoriaRepository extends JpaRepository<Subcategoria, Long> {
    List<Subcategoria> findByNomeContainingIgnoreCase(String nome);
}
```

---

### 5. **Serviço**

```java
package com.example.demo.service;

import com.example.demo.dto.SubcategoriaDTO;
import com.example.demo.dto.SubcategoriaRequestDTO;
import com.example.demo.mapper.SubcategoriaMapper;
import com.example.demo.model.Subcategoria;
import com.example.demo.repository.SubcategoriaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class SubcategoriaService {

    @Autowired
    private SubcategoriaRepository subcategoriaRepository;

    @Autowired
    private SubcategoriaMapper subcategoriaMapper;

    public List<SubcategoriaDTO> listarTodas() {
        return subcategoriaRepository.findAll().stream()
                .map(subcategoriaMapper::toDTO)
                .collect(Collectors.toList());
    }

    public Page<SubcategoriaDTO> listarComPaginacao(Pageable pageable) {
        return subcategoriaRepository.findAll(pageable)
                .map(subcategoriaMapper::toDTO);
    }

    public SubcategoriaDTO buscarPorId(Long id) {
        Subcategoria subcategoria = subcategoriaRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Subcategoria não encontrada para o ID: " + id));
        return subcategoriaMapper.toDTO(subcategoria);
    }

    public List<SubcategoriaDTO> buscarPorNome(String nome) {
        List<Subcategoria> subcategorias = subcategoriaRepository.findByNomeContainingIgnoreCase(nome);
        if (subcategorias.isEmpty()) {
            throw new RuntimeException("Nenhuma subcategoria encontrada com o nome: " + nome);
        }
        return subcategorias.stream()
                .map(subcategoriaMapper::toDTO)
                .collect(Collectors.toList());
    }

    public SubcategoriaDTO criar(SubcategoriaRequestDTO dto) {
        Subcategoria subcategoria = subcategoriaMapper.toEntity(dto);
        return subcategoriaMapper.toDTO(subcategoriaRepository.save(subcategoria));
    }

    public SubcategoriaDTO atualizar(Long id, SubcategoriaRequestDTO dto) {
        Subcategoria existente = subcategoriaRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Subcategoria não encontrada para o ID: " + id));
        existente.setNome(dto.getNome());
        return subcategoriaMapper.toDTO(subcategoriaRepository.save(existente));
    }

    public void deletar(Long id) {
        if (!subcategoriaRepository.existsById(id)) {
            throw new RuntimeException("Subcategoria não encontrada para o ID: " + id);
        }
        subcategoriaRepository.deleteById(id);
    }
}
```

---

### 6. **Controller**

```java
package com.example.demo.controller;

import com.example.demo.dto.SubcategoriaDTO;
import com.example.demo.dto.SubcategoriaRequestDTO;
import com.example.demo.service.SubcategoriaService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.List;

@RestController
@RequestMapping("/api/subcategorias")
public class SubcategoriaController {

    @Autowired
    private SubcategoriaService subcategoriaService;

    @GetMapping
    public ResponseEntity<List<SubcategoriaDTO>> listarTodas() {
        List<SubcategoriaDTO> subcategorias = subcategoriaService.listarTodas();
        if (subcategorias.isEmpty()) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.ok(subcategorias);
    }

    @GetMapping("/paginadas")
    public ResponseEntity<Page<SubcategoriaDTO>> listarComPaginacao(Pageable pageable) {
        Page<SubcategoriaDTO> subcategorias = subcategoriaService.listarComPaginacao(pageable);
        return ResponseEntity.ok(subcategorias);
    }

    @GetMapping("/{id}")
    public ResponseEntity<SubcategoriaDTO> buscarPorId(@PathVariable Long id) {
        try {
            return ResponseEntity.ok(subcategoriaService.buscarPorId(id));
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }

    @GetMapping("/buscar-por-nome")
    public ResponseEntity<List<SubcategoriaDTO>> buscarPorNome(@RequestParam String nome) {
        try {
            return ResponseEntity.ok(subcategoriaService.buscarPorNome(nome));
        } catch (RuntimeException e) {
            return ResponseEntity.noContent().build();
        }
    }

    @PostMapping
    public ResponseEntity<SubcategoriaDTO> criar(@Valid @RequestBody SubcategoriaRequestDTO dto) {
        return ResponseEntity.ok(subcategoriaService.criar(dto));
    }

    @PutMapping("/{id}")
    public ResponseEntity<SubcategoriaDTO> atualizar(@PathVariable Long id, @Valid @RequestBody SubcategoriaRequestDTO dto) {
        try {
            return ResponseEntity.ok(subcategoriaService.atualizar(id, dto));
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletar(@PathVariable Long id) {
        try {
            subcategoriaService.deletar(id);
            return ResponseEntity.noContent().build();
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }
}
```

---

### 7. **Global Exception Handler (Opcional)**

Crie um **`@ControllerAdvice`** para tratamento global de erros.

```java
package com.example.demo.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<String> handleRuntimeException(RuntimeException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body("Erro: " + e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleException(Exception e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Erro interno: " + e.getMessage());
    }
}
```

---

Essa é a implementação completa, moderna e alinhada com práticas de mercado, incluindo DTOs, validações e um mapeador. Essa estrutura é modular e escalável.
